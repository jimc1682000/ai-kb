# AI 驅動的 DevOps 知識庫系統 — 完整說明

## 設計動機

### 文件庫 vs 知識庫

傳統文件庫（如 Confluence、Wiki）的問題：
- 靜態文件、樹狀結構，寫了就沒人更新
- 搜尋靠關鍵字，找不到就放棄
- 資訊過時比沒有資訊更危險

AI 驅動的知識庫：
- 原子化筆記、網狀連結（wiki-link）
- AI 持續精煉、自動偵測過時內容
- 語意搜尋，用自然語言問答

### 團隊痛點

1. 知識散落各處（Slack、wiki、個人筆記），找不到
2. 找不到就重新踩坑，浪費時間
3. 傳統 wiki 維護成本高，沒人更新

### 設計理念

借鑑卡片盒筆記法（Zettelkasten）的核心原則：
- 原子化：每篇筆記聚焦一個主題
- 互連：AI 自動建立文件之間的連結
- 漸進累積：增量處理，每次只精煉新內容

核心思想：人只負責「記下來」，AI 負責「讓它有用」。

## 三層架構

### Layer 1: Raw Notes（原始筆記）

- 團隊成員用 Markdown 寫筆記，每人一個資料夾
- 簡單 frontmatter：author, tags, noted_at
- 5 種模板：troubleshooting / architecture / howto / reference / general
- 零門檻：不要求排版完美、不要求分類正確
- 用 `/kb-new` 指令，AI 自動判斷模板、問答補齊資訊

### Layer 2: AI Digest（精煉層）

- AI Skill（`/kb-digest`）將原始筆記精煉為結構化知識文件
- 獨立 Git repo，扁平結構（同時是 Obsidian vault）
- 自動建立 wiki-link 連結相關主題（Obsidian graph view）
- 交叉比對 IaC 程式碼（Terraform/Serverless）驗證事實
- 處理矛盾：不同人對同一主題有不同說法時，兩邊都保留並標註來源
- 標記過時資訊
- 每個檔案控制在 300 行以內，超過自動拆分
- 分類用 frontmatter：categories（技術領域）、types（文件類型）、tags（細項）

### Layer 3: NotebookLM（語意問答）

- `/kb-publish` 按 category 合併精煉文件，上傳到 Google NotebookLM
- 任何人都可以用自然語言問問題，AI 從知識庫回答
- 支援 Deep Research（搜尋網路延伸 source 知識）
- 多元輸出：語音摘要、簡報、報告、心智圖、學習卡、測驗
- 非技術人員也能使用

### 額外：Library（TBD）

分為三個區域：
- _inbox：待處理文件
- shared-docs：跨團隊共享文件
- team-docs：團隊工作文件
- 也是 Obsidian vault
- 用 `/kb-publish --target library` 上傳到另一個 NotebookLM notebook

## AI Skill 自動化流程

系統使用 6 個 Claude Code Skill 自動化整條流水線：

### /kb-search — 遞迴搜尋（RLM 風格）

知識庫的查詢入口，借鑑 RLM（Recursive Language Models）概念：
- Progressive Collection Search：先搜 digest 精煉文件 → 再深入 raw notes / Library / IaC
- qmd 混合搜尋引擎：BM25 關鍵字 + Vector 語意 + LLM Reranker 三層融合
- Query 意圖分類：聚焦 query 直接搜、廣泛 query 先搜再問方向、模糊 query 先問再搜
- LLM 遞迴決策：每輪搜尋後自主決定下一步（搜 raw、查 IaC、展開 wiki-link、問使用者）
- --quick 模式：BM25 全掃秒級回答

專案啟動建議：新專案開始前，用 /kb-search 搜尋相關知識，再手動把需要的檔案複製到專案 docs/ 作為 reference。

### /kb-new — 寫一張新卡片

遵循卡片盒筆記法的 append-only 原則——永遠寫新卡片，不修改既有卡片：
- 寫之前先搜 KB 找相關卡片，自動建立 wiki-link（續卡、修正、分支）
- 判斷最適合的模板類型
- 產生語義化檔名（kebab-case）
- 缺少資訊時一次問完所有問題
- 寫入完整筆記（不留佔位符）
- 自動 git commit + push

### /kb-digest — AI 精煉（核心引擎）

增量處理原始筆記，產出精煉文件：
- 只處理上次之後的新增/修改筆記（git diff）
- 整合多人對同一主題的筆記（重複偵測）
- 建立 wiki-link 連結相關主題
- 交叉比對 5 個 IaC repo 驗證事實
- 矛盾處理：時間差異 → 時間線敘述；觀點差異 → 兩邊保留標註來源
- 過時標記：筆記描述的設定已在 IaC 中移除時
- 300 行上限：超過自動拆分，用連結串起來
- 敏感資訊掃描：發現密碼/token → 阻塞 commit
- 不確定性保留：原始筆記的「可能」「疑似」「不確定」等語氣在精煉中保留，不被確定化
- 異常信號保留：看似離題但不尋常的觀察，用旁註格式保留
- confidence 欄位：每篇精煉文件標記確定性等級（single-observation / multiple-observations / iac-verified）
- 反框架檢查：新筆記挑戰既有 digest 的假設時，標記分歧而非靜默合併
- 合併比例追蹤（merge_ratio）：合併進既有檔案 vs 獨立建檔的比例，偏高時警告可能有詮釋循環

### /kb-publish — 上傳 NotebookLM

合併精煉文件後上傳到 Google NotebookLM：
- 按 categories 合併（每個 category 一個 source）
- 移除 wiki-link 語法（NLM 不認得）
- 檢查字元數（超過 500,000 字元 → 阻塞，建議拆分 category）
- 上傳前敏感資訊掃描（阻塞門檻）
- Diff-based：只重新上傳有變更的 category
- NLM 不支援更新 → 刪除舊 source + 重新上傳
- 支援 test / prod 環境選擇
- 上傳後結構性驗證（advisory）
- 元資料保留：合併時保留 last_updated 和 confidence 為可讀文字，NLM 回答可標註時效性和確定性

### /kb-verify — 品質驗證

多階段驗證，可獨立執行。報告分為格式合規和內容品質兩個獨立判定：

格式合規：
- 敏感資訊掃描（密碼、API key、token、AWS key）— Blocker
- Frontmatter 完整性 — Advisory
- Wiki-link 孤立連結偵測 — Advisory
- NLM source 數量比對 — Advisory

內容品質（抽查）：
- 隨機抽 2-3 篇 digest 比對 raw note，檢查不確定性有沒有被抹平 — Advisory
- confidence 欄位與 sources 數量是否一致 — Advisory
- NLM 正向 smoke test（已知答案 query） — Advisory
- NLM 負向 smoke test（不該有答案的 query，偵測幻覺風險） — Advisory

### /kb-feedback — 知識缺口分析

AI 主動分析知識缺口，產出 TODO 建議報告：
- 維度 1：IaC 無文件（有 Terraform module 但沒有對應筆記）— 高優先
- 維度 2：淺層主題（某 tag 只有 1-2 篇）— 中優先
- 維度 3：過時風險（筆記超過 6 個月沒更新）
- 維度 4：缺特定文件類型（有 architecture 但沒有 troubleshooting）— 低優先
- 維度 5：孤立 wiki-link（指向不存在的文件）
- 與上次報告比較：標記新增/持續/已解決

## Skill 安裝方式

Skill 透過 npx skills（vercel-labs/skills）管理：

```
npx skills add <source> --skill <name> -a claude-code -g --copy
npx skills list                  # 列出已安裝的 skill
npx skills update                # 更新所有 skill 到最新版
```

每個 skill 是一個 SKILL.md 定義檔，描述 AI 的行為、可用工具、和執行步驟。
安裝後在 Claude Code 中用 `/skill-name` 呼叫。

## 安全設計

### 兩道敏感資訊掃描

1. /kb-digest 產出時掃描 → 有敏感資訊？→ 阻塞 commit
2. /kb-publish 上傳前掃描 → 有敏感資訊？→ 阻塞上傳

兩道都是 blocker（不是 warning）。

掃描內容：密碼、API key、token、private key、AWS access key、資料庫連線字串、內部 IP

## 實際數據

- 約 100 篇原始筆記，2 位作者
- 約 70 篇精煉文件，7 個技術領域
- 5 個 IaC repo 交叉參照
- NotebookLM 22 個 sources（上限 50）
- 系統設計 + Skill 開發約 3 週
- 維護成本接近零（AI 自動精煉 + 上傳）

## NotebookLM 的限制

- 每個 notebook 最多 50 個 sources（Free tier；付費版可達 300）
- 每個 source 最多 500,000 字元
- 不支援更新 API（只能刪除 + 重新建立）
- 不支援嵌入圖片

## 工具鏈

- Claude Code：AI coding agent，執行 skill 的 runtime
- npx skills（vercel-labs/skills）：skill 套件管理
- nlm CLI（notebooklm-mcp-cli）：NotebookLM API 操作，支援 MCP 整合
- Obsidian：知識庫閱覽器（vault + graph view + wiki-link）
- Git：版本控制
- Mermaid：架構圖繪製
