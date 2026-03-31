# 三層知識庫架構

![[architecture-diagram]]

## Layer 1: Raw Notes — 原始筆記

**誰寫**：團隊成員（人）
**放哪裡**：Git repo，每人一個資料夾
**格式**：Markdown + 簡單 frontmatter（author, tags）

```yaml
---
author: member-a
tags: [troubleshooting, redis, timeout]
noted_at: 2026-01-15
---
# Redis Timeout 問題排查
...
```

**設計原則**：
- 零門檻：只要會寫 markdown 就行
- 不要求排版完美、不要求分類正確
- 有 5 種模板可選（troubleshooting / architecture / howto / reference / general）
- 用 `/kb-new` 指令，連模板都不用自己選

## Layer 2: AI Digest — 精煉層

**誰寫**：AI Skill（`/kb-digest`）
**放哪裡**：獨立 Git repo，扁平結構（Obsidian vault）
**格式**：精煉後的 Markdown + wiki-link 互連

**AI 做了什麼**：
- 整合多人對同一主題的筆記
- 建立 `[[wiki-link]]` 連結相關主題（→ Obsidian graph view）
- 交叉比對 IaC 程式碼驗證事實
- 處理矛盾（不同人的不同觀察）
- 標記過時資訊
- 每個檔案控制在 300 行以內，太長自動拆分

**分類方式**：不用資料夾，用 frontmatter：
- `categories`：技術領域（redis, ecs, networking...）
- `types`：文件類型（troubleshooting, architecture, howto...）
- `tags`：細項標籤

**為什麼用 Obsidian？** 因為它不擋路：
- **Local-first + 純 Markdown**：檔案就是檔案，Git 直接管，沒有 vendor lock-in
- **`[[wiki-link]]` 原生支援**：AI 建的連結直接可點擊，不用額外處理
- **Graph view**：知識網狀連結的視覺化，一眼看出哪些主題互連
- Vault 就是一個資料夾——換工具零成本

## Layer 3: NotebookLM — 語意問答

**誰用**：所有人（包含非寫筆記的人）
**怎麼用**：自然語言提問，AI 從知識庫回答

**流程**：
1. `/kb-publish` 按 category 合併精煉文件
2. 上傳到 Google NotebookLM
3. 任何人都可以用中文問問題

→ 詳見 [[notebooklm-integration]]

## 額外：Library（TBD）

除了 AI 精煉，還有一個 Library repo，分為三個區域：
- **_inbox**：待處理文件
- **shared-docs**：跨團隊共享文件
- **team-docs**：團隊工作文件

Library 也是 Obsidian vault，也可以上傳到 NotebookLM（`/kb-publish --target library`）。

## 跨層查詢：/kb-search

除了寫入流（kb-new → kb-digest → kb-publish），還有一條**查詢流**：

`/kb-search` 不侷限於單一層，而是遞迴跨層搜尋：
1. 先搜 **Digest**（精煉文件，快速定位）
2. 再深入 **Raw Notes**（原始筆記，完整上下文）
3. 必要時查 **IaC** 和 **Library**

→ 搜尋技術細節見 [[search-concepts]]

## 為什麼是三層不是一層？

| 需求 | 一層能滿足嗎？ |
|------|--------------|
| 快速記錄 | ✅ 可以 |
| 知識互連 | ❌ 人不會主動建連結 |
| 語意搜尋 | ❌ 原始筆記太散 |
| 品質控管 | ❌ 沒有驗證機制 |
| 非作者也能受益 | ❌ 別人的筆記看不懂 |

三層各司其職：**人負責輸入，AI 負責加工，NLM 負責輸出**。

→ AI 怎麼做到的？[[ai-skills]]
→ 回到 [[index]]
