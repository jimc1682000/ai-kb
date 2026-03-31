# 演講劇本

> **題目**：去找吧！我把所有的一切都放在那裡了！
> **時間**：20 分鐘
> **工具**：Obsidian（主軸）+ NotebookLM（交錯 demo）

---

## 時間線

```
0:00  [Obsidian] 開場 index.md → 快速帶過
0:01  [NLM] 鬼切啟動 Deep Research → 切回
0:02  [Obsidian] motivation（停久一點，帶痛點共鳴）
0:05  [Obsidian] zettelkasten → knowledge-scope → architecture → search-concepts + graph view
0:08  [NLM] Deep Research 進度 → import → 卡片盒筆記法 demo
0:10  [Obsidian] ai-skills.md（知識旅程 + 6 skills）
0:13  [NLM] KB sources + live demo
0:18  [NLM] 現場觀眾問題 live demo
0:20  收尾
```

### Deep Research 實測時間

| 啟動 → 研究完成 | import 82 sources | 總計 |
|-----------------|-------------------|------|
| ~4 分鐘 | ~1 分 48 秒 | **~6 分鐘** |

> 0:01 啟動 → 講 motivation+architecture → 0:08 回來剛好完成

---

## 0:00 — 開場

**打開 `index.md`** — 快速帶過標題和大綱，不停留

- 「今天分享我們怎麼用 AI 解決知識管理的問題」

## 0:01 — 啟動 Deep Research

**切 NLM → 啟動 Deep Research「卡片盒筆記法(Zettelkasten) AI」**

- 「先讓大家看 Google NotebookLM——它讀你的資料，回答問題」
- 「AI 花一點時間研究，我們先繼續」

**切回 Obsidian**

## 0:02 — 動機（多停一下，讓觀眾有共鳴）

**帶 `motivation.md`**
- 三個痛點逐一帶，讓觀眾點頭：翻六七個工具、反覆踩坑+過期文件、寫文件好麻煩
- 文件庫 vs 知識庫表格——強調「零摩擦輸入」
- 「可是知識還在 Jira / Confluence 裡啊？」→ 不用遷移，遇到再寫

## 0:05 — 方法論 → 知識範圍 → 架構 → 搜尋演化

**帶 `zettelkasten.md`** → 三原則（原子化、互連、漸進累積）
- 「不是發明新東西——這套方法讓 Luhmann 寫了 70 本書」
- 精煉流程圖：閃念筆記 → 文獻筆記 → 永久筆記
- 「瓶頸是建連結的人力成本，AI 解決這個」

**帶 `knowledge-scope-diagram.md`** → 三個來源的交集
- 「Raw Notes + Library + IaC Repos 交叉比對，產出精煉知識」

**帶 `architecture.md`** → 三層架構圖 → 「人負責輸入，AI 負責加工，NLM 負責輸出」

**帶 `search-concepts.md`** → 演化光譜圖
- 「搜尋技術從 Naive RAG 到 RLM，我們在中間偏右」
- 「大顆粒用 RAG 加速定位，小顆粒用 RLM 精神深入」

**切 graph view**
- 「每個節點是一份文件，連線是 wiki-link，AI 精煉時自動建立」

## 0:08 — 回到 NLM

**切 NLM，確認 Deep Research 狀態**

> 還沒好 → 「還在跑，我們先繼續」→ 跳回 ai-skills
> 完成但要 import → 按 import，口頭填：「AI 幫我們從網路上找了 80+ 篇資料」

### Demo 完成後：

**Q0「誰是王建民？」** → NLM 答不了
- 「**不知道就說不知道**——只根據你給的資料回答，不會幻覺」

**Q1「除了卡片盒筆記法，還有什麼筆記法？」** → 展示 deep research 廣度

**Q2「有了 AI 之後的卡片盒筆記法有什麼變化嗎？」**
- 「這就是我們系統的設計理念——AI 自動化最花時間的部分：建連結和持續更新」

## 0:10 — AI Skills

**帶 `ai-skills.md`**

- 6 個 skill 快速過，重點講 `/kb-search`、`/kb-new`、`/kb-digest`
- `/kb-search` 亮點：
  - 「新專案啟動時，先用 kb-search 拉相關知識——不用自己翻六七個工具」
- `/kb-new` 亮點：
  - 「永遠寫新卡片，不改舊的——卡片盒筆記法的 append-only 原則」
  - 「寫之前會搜 KB 找相關卡片，自動建 link」
- `/kb-search` 亮點：
  - 「不是 grep，是 AI 自主搜尋——先搜精煉文件定位，再遞迴深入原始筆記和 IaC 程式碼」
  - 「廣泛問題會先列出方向讓你選，不會一股腦全倒出來」
- `/kb-digest` 亮點：
  - IaC 交叉驗證、矛盾處理
  - **不確定性保留**：「AI 不會把『可能是 X』改成『確定是 X』——原始的猶豫和條件限定都保留」
  - **confidence 欄位**：「每篇精煉文件標記確定性——單次觀察、多次觀察、還是 IaC 佐證」
- `/kb-publish` 亮點：
  - 「合併上傳 NLM 時**保留日期和確定性**，NLM 回答就能說『根據 2026-03 的觀察』而非一概而論」
- `/kb-verify` 亮點：
  - 「不只檢查格式，還會**抽查內容品質**——隨機抽幾篇 digest 比對 raw note，看有沒有把不確定的東西說死」
  - 「還有**負向 smoke test**——故意問 NLM 不該知道的問題，看它會不會幻覺」
- 端到端流程 → 卡片盒三原則對應
- skill 安裝方式（`npx skills`）

## 0:13 — KB Demo

**切 NLM → 加 source-kb-system.md + source-demo-content.md**

**Q1「這個知識庫系統的整體架構是什麼？」**
- 帶出三層架構，跟剛才 Obsidian 對照

**Q2「AI Skill 在這個系統中負責哪些工作？」**
- 帶出 6 個 skill

**Q3「Redis 之前出過什麼問題？怎麼解決的？」**
- 「**這就是知識庫的價值**——30 秒有答案，不用翻 Slack」

**Q4「這個系統如何確保不會洩漏敏感資訊？」**
- 帶出兩道掃描閘門

## 0:18 — 現場問答 live demo

現場收 2-3 個觀眾問題，直接問 NLM

- 答得好 → 「知識庫有涵蓋」
- 答不好 → 「這就是 /kb-feedback——AI 自動偵測知識缺口」

## 0:20 — 收尾（帶 `takeaway.md`）

- 「三件事帶走」
  1. 「瓶頸不是沒人寫，是寫了之後的事——AI 接手連結和更新」
  2. 「AI 只是輔助——建連結、精煉、偵測過時都交給它，但最有價值的是你的第一手觀察」
  3. 「永遠寫新卡片——知識的演變本身就有價值」
- 「想開始？一個 Git repo + 一份 markdown，就這樣」
- 「有興趣歡迎來聊」

---

## 備用

| 狀況 | 應對 |
|------|------|
| **時間不夠** | 跳過卡片盒 demo（Q1, Q2），直接 KB demo |
| **時間太多** | 展開 `zettelkasten.md`、`search-concepts.md` 或 `toolchain.md` |
| **NLM 掛了** | 全程 Obsidian，口頭描述 NLM，展示 backup-answers.md |
| **Deep search 太慢** | 不等，直接進 KB demo |

## 備用觀眾問題

1. 「ECS 的設定檔是怎麼注入到 container 的？」
2. 「JVM OOM 的時候怎麼判斷是不是 memory leak？」
3. 「IAM 權限變更時要注意什麼？」
