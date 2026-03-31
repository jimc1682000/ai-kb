# DevOps 知識庫 — 技術知識範例

以下是知識庫中的實際精煉文件範例，展示不同類型的知識內容。

---

## Redis Timeout 問題排查與解決

### 問題描述

- **服務**：後端 API 服務（Java, Spring Boot, ECS Fargate）
- **症狀**：每小時 cache refresh 或 Scheduled Scaling scale-out 時，約 5% request error
- **錯誤訊息**：`RedisCommandTimeoutException: Command timed out after 1 second(s)`
- **觸發時機**：每小時 cache refresh job 執行、新 ECS Task 啟動時、Rolling Deployment 冷啟動

### 根因分析：G1GC Stop-The-World (STW) Pauses

ElastiCache 監控顯示尖峰 Engine CPU 約 37%，排除 Redis server 瓶頸。問題在 JVM 端。

觸發流程：
1. 每小時 cache refresh job 執行
2. 大量物件分配（DB fetch、LZ4 序列化、cache entry 建立）
3. G1GC Young Gen 填滿，觸發 Evacuation Pause (STW)
4. 所有 JVM thread 凍結（包含 Netty EventLoop）
5. Redis server response 已到達 network buffer，但 Netty 沒有 CPU 時間處理
6. Lettuce command timeout (1s) 到期 → 拋出 RedisCommandTimeoutException

### commandTimeout 設定 Bug

在調低 `redis_pool_max_wait_millis` 從 2000 到 500 時，發現 application code 的 `LettuceConnectionFactory` 直接用 `maxWaitMillis` 作為 `commandTimeout`：

| Timeout 類型 | 用途 | 合理範圍 |
|-------------|------|---------|
| `maxWaitMillis` | 從 pool 借 connection 的等待時間 | 500ms - 2000ms |
| `commandTimeout` | Redis 指令執行 timeout | 2000ms - 5000ms |

兩者不應共用同一個值。

### 解決方案

**已實施：shareNativeConnection=false（緩解）**

Lettuce 預設 `shareNativeConnection=true`，所有 Tomcat thread 共用單一 TCP connection。改為 false 後，command 分散到多條 connection，GC STW 後各 connection 的 timeout 預算不同，同時 timeout 的機率大幅降低。

測試數據：錯誤從 ~5% 降到 <1%，部分場景降到 0。

**根本解決：Application-level Warmup**

Spring Boot Health Check 通過前，先完成 JIT 編譯和 Connection Pool 預熱。透過 `ApplicationRunner` 主動發起 dummy API call。

**已排除方案：放寬 Redis Timeout**

增加 timeout 反而更糟——所有 Tomcat thread 一起等，p90 暴增，累積的 thread 造成雪球效應。

### 監控指標

| 指標 | 來源 | 告警門檻 |
|------|------|---------|
| HTTPCode_Target_5XX_Count | CloudWatch ALB | 0 |
| EngineCPUUtilization | CloudWatch ElastiCache | < 50% |
| p90 latency | APM | < 2s |
| Redis Pool Exhaustion Error | Application Logs | 0 |

---

## ECS Configuration Injection 架構決策紀錄

### 背景

評估 AWS ECS 原生設定管理方案，決定最適合現有運維狀況的注入方式。

### 評估方案

**方案一：AWS AppConfig（現代動態設定）**

架構：Secrets Manager / S3 → AWS AppConfig → AppConfig Agent (Sidecar) → Application

優點：
- 動態更新：不重啟 Task 即可透明 refresh
- 驗證機制：支援 Lambda / JSON Schema 預部署攔截
- 部署策略：支援 Canary（10% → 50%）+ Alarm 自動 rollback

**方案二：ECS Storage for S3（原生 Mount）**

利用 AWS Mountpoint for Amazon S3 做 cloud-native mount。
- 零程式碼變更：Application 直接從路徑讀取
- 多檔案支援：適合高度模組化的設定結構

### 實際限制

| 服務 | 設定檔數量 | 關鍵特性 |
|------|-----------|---------|
| 後端 API | 5+ | GeoIP 二進位檔、XML、YAML、logback |
| 資料服務 | 4-6 | Firebase JSON、多 Spring Profile |
| Reverse Proxy | 10+ | 高度模組化、import entity path 依賴 |

AppConfig 不擅長 binary 檔案和多檔案 import 語法，強行整合需要大量改寫。

### 決策結論

**維持現有方案（s5cmd + S3），搭配 Secrets Manager 強化敏感資料安全。**

理由：不為「技術美學」引入與現有多檔案結構不相容的複雜架構。

短期優化：敏感變數遷移到 AWS Secrets Manager，透過 Task Definition 環境變數原生注入。

---

## 後端服務 OOM 分析：JVM 參數設定瓶頸

### 問題描述

- **服務**：後端 API（AWS ECS Fargate, 5GB RAM → 後調整為 7GB）
- **結論**：非 code leak，確認為 JVM 參數設定瓶頸導致吞吐與回收速度不平衡

### 事件時間軸

**Phase 1: 潛伏期** — 記憶體 ~65%，GC 懶得收。手動 `jcmd GC.class_histogram` 後 Heap 從 ~3.2GB 降到 ~2.2GB，確認有 1GB+ 可回收垃圾。

**Phase 2: 健康擴張** — 流量上升，Young Gen 擴張到 1.33GB（佔 Heap 56%）。正常。

**Phase 3: 危險黃金交叉** — Old Gen 累積（翻倍到 1.16GB），Young Gen 被迫犧牲（減半到 0.65GB）。IHOP Trap：Old Gen 佔 41%（< 45% 門檻），JVM 認為「不需要背景回收」。

**Phase 4: 窒息崩潰** — 記憶體 92.5%，p90 12.2 秒，CPU 100%（GC 掙扎）。Young Gen 剩 241MB，Off-Heap 從 0.74GB 暴增到 1.54GB。

**Phase 5: 死裡逃生** — Full GC 從 2.33GB 清到 1.36GB（清掉 ~1GB），再次證明不是 Memory Leak。

### 主因與併發症

**主因：GC 設定瓶頸**
- Container 太小：`MaxRAMPercentage=60` 導致 Heap 上限 3GB
- 排水太慢：`IHOP=45` 在 Old Gen 41% 時不啟動回收

**併發症：Off-Heap 暴增**
- R-Sets 擴張：GC 掙扎導致 G1 建立大量 Remembered Sets
- Direct Buffer 累積：GC 停滯導致持有 Direct Buffer 的物件無法回收

### 解決方案

| 參數 | 原值 | 新值 | 目的 |
|------|------|------|------|
| InitiatingHeapOccupancyPercent | 45 | 35 | 提前啟動背景回收 |
| G1ReservePercent | 10 | 15 | 增加大物件容錯 |
| MaxDirectMemorySize | 未設定 | 1024m | 防止 Off-Heap 超過 container 限制 |

最終調整：Fargate Memory 從 5GB 提升到 7GB，峰值用量回到 ~83%。

---

## IAM 權限管理操作原則

### 核心原則：先建後拆、先接後斷

變更 IAM 權限時，必須確保使用者在任何時刻都不會失去必要的存取權限。

### 標準操作順序

```
1. 建立新 Policy → 確認內容正確
2. Attach 新 Policy → 使用者同時擁有新舊權限
3. Detach 舊 Policy → 移除不再需要的權限
4. (可選) 刪除舊 Policy
```

### 為什麼不能反過來

先 Detach 再 Attach 會造成權限真空期：
- 進行中的操作可能立即失敗（S3 上傳中斷、API 呼叫被拒絕）
- 即使只有幾秒，也可能造成資料不一致

### 同樣原則適用於

| 場景 | 操作順序 |
|------|---------|
| IAM Role Policy 替換 | 先 Attach 新 → 再 Detach 舊 |
| Security Group Rule 變更 | 先加新規則 → 確認連通 → 再移舊規則 |
| DNS 切換 | 先建新記錄 → 確認解析 → 再刪舊記錄 |
| ALB Target Group | 先註冊新 target → Health check 通過 → 再移舊 target |

### 操作清單

- 操作前用 `aws iam list-attached-user-policies` 確認目前 attached policies
- 新 policy 建立後用 `get-policy-version` review 確認內容
- 如果是收窄權限（縮小範圍），確認使用者不再需要舊權限
- 如果舊 policy 有其他使用者 attached，只 detach，不 delete
