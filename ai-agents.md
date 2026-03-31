# AI Agent 自動化流程

## 什麼是 Skill？

Skill 是 Claude Code 的擴充功能——一個 `SKILL.md` 定義檔，描述 AI agent 的行為、可用工具、和執行步驟。

安裝後用 `/skill-name` 呼叫，就像 CLI 指令一樣。

### 安裝方式

```bash
# 從本地 repo 安裝（會複製到 ~/.claude/skills/）
npx skills add /path/to/repo

# 列出已安裝的 skill
npx skills list

# 更新（重新安裝）
npx skills add /path/to/repo
```

> `npx skills` 來自 [vercel-labs/skills](https://github.com/vercel-labs/skills)，是跨多種 AI Agent 的 skill 套件管理工具。

### Skill 的結構

```
repo/.claude/skills/
└── kb-digest/
    └── SKILL.md    ← AI agent 的「劇本」
```

```yaml
---
name: kb-digest
description: 精煉原始筆記到 digest repo
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# 執行步驟
## Step 1: 準備工作區
...
```

## 6 個 Knowledge Base Skills

### `/kb-search` — 遞迴搜尋

知識庫的查詢入口。借鑑 RLM（Recursive Language Models）概念，LLM 自主判斷相關性、決定往哪裡深入。

- **Progressive Collection Search**：先搜 digest 取得結構化概覽 → 再深入 raw notes / Library / IaC
- **qmd 混合引擎**：BM25 關鍵字 + Vector 語意 + LLM Reranker 三層融合
- **Query 意圖分類**：
  - 聚焦 query（如「redis timeout」）→ 直接搜
  - 廣泛 query（如「jenkins」）→ 先搜，列出主題方向讓使用者選
  - 模糊 query（如「deployment」）→ 先問使用者意圖再搜
- **LLM 遞迴決策**：每輪搜尋後 LLM 決定下一步——搜 raw、查 IaC、展開 wiki-link、還是詢問使用者
- **--quick 模式**：BM25 全掃，秒級回答

### `/kb-new` — 快速建立筆記

```
使用者：「記一下 Redis timeout 的問題」
AI：判斷 → troubleshooting 模板 → 問答補齊 → 寫入完整筆記 → git commit & push
```

- AI 自動判斷模板類型（troubleshooting / architecture / howto / reference / general）
- 產生語義化檔名（`redis-connection-timeout.md`）
- 缺少資訊時一次問完，不逐一問
- 寫入後自動 commit + push

### `/kb-digest` — AI 精煉

知識庫的核心引擎。把原始筆記轉化為結構化、互連的知識文件。

- **增量處理**：只處理上次之後的新增/修改筆記
- **IaC 交叉驗證**：筆記說 timeout 是 1 秒，去 Terraform 確認實際值
- **矛盾處理**：不同人對同一主題有不同說法？兩邊都保留，標註來源
- **自動 wiki-link**：建立 `[[相關主題]]` 連結
- **300 行上限**：太長自動拆分，用連結串起來
- **敏感掃描**：發現密碼/token → 阻塞 commit
- **不確定性保留**：原始筆記中的「可能」「疑似」「不確定」不會在精煉中被確定化
- **異常信號保留**：看似離題但不尋常的觀察，用旁註保留（可能是其他問題的線索）
- **confidence 欄位**：每篇精煉文件標記確定性等級（單次觀察 / 多次觀察 / IaC 佐證）
- **反框架檢查**：新筆記挑戰既有 digest 的假設時，標記分歧而非靜默合併

### `/kb-publish` — 上傳 NotebookLM

把精煉文件合併後上傳到 Google NotebookLM。

- 按 `categories` 合併（每個 category 一個 source）
- 上傳前敏感掃描（阻塞門檻）
- **Diff-based**：只重新上傳有變更的 category
- NLM 不支援「更新」→ 刪除舊 source + 重新上傳
- 支援 test / prod 環境選擇
- **元資料保留**：合併時保留日期和確定性等級，NLM 回答可標註時效性和可信度

### `/kb-verify` — 品質驗證

多階段驗證，分為**格式合規**和**內容品質**兩個獨立判定：

**格式合規**：
- 敏感資訊掃描（密碼、API key、token）
- Frontmatter 完整性
- Wiki-link 孤立連結偵測
- NLM source 數量比對

**內容品質（抽查）**：
- 隨機抽 2-3 篇 digest 比對 raw note，檢查不確定性有沒有被抹平
- confidence 欄位與 sources 數量是否一致
- NLM 正向 smoke test（已知答案 query）
- NLM 負向 smoke test（不該有答案的 query → 偵測幻覺風險）

### `/kb-feedback` — 知識缺口分析

AI 主動告訴你「你們缺什麼文件」：

| 維度 | 分析方法 |
|------|----------|
| IaC 無文件 | 有 Terraform module 但沒有對應筆記 |
| 淺層主題 | 某個 tag 只有 1-2 篇，內容偏薄 |
| 過時風險 | 筆記超過 6 個月沒更新 |
| 缺特定類型 | 有 architecture 但沒有 troubleshooting |
| 孤立連結 | wiki-link 指向不存在的文件 |

產出 TODO checklist，團隊成員認領補充。

## 端到端流程

```
/kb-new → 寫筆記（原子化：一篇聚焦一個主題）
           ↓
/kb-digest → 精煉 + 連結（互連：AI 自動建立 wiki-link）
           ↓
/kb-verify → 品質檢查（格式合規 + 內容品質抽查）
           ↓
/kb-publish → 上傳 NotebookLM
           ↓
/kb-feedback → 分析缺口 → 回到 /kb-new（漸進累積：循環補齊知識）

/kb-search → 隨時可查（RLM 遞迴搜尋，digest → raw → IaC → Library）
```

這個循環對應 [[zettelkasten|Zettelkasten 卡片盒筆記法]]的三個核心原則：**原子化、互連、漸進累積**。差別在於，傳統 Zettelkasten 靠人力建連結，我們讓 AI 接手這些重複勞動。

→ NotebookLM 怎麼整合的？[[notebooklm-integration]]
→ 安全怎麼做的？[[security]]
→ 回到 [[index]]
