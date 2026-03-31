# 備援回答

> NLM 掛掉時用。每題的文字版回答。

## 「這個知識庫系統的整體架構是什麼？」

三層架構：

1. **Raw Notes**：團隊成員寫 Markdown 筆記，每人一個資料夾，零門檻
2. **AI Digest**：AI Skill 精煉成結構化文件，自動建立 wiki-link，是 Obsidian vault
3. **NotebookLM**：合併上傳，語意問答，非技術人員也能用

額外有 Library 層：_inbox（待處理文件）、shared-docs（跨團隊共享文件）、team-docs（團隊工作文件）。

## 「AI Skill 在這個系統中負責哪些工作？」

6 個 Claude Code Skill：
- `/kb-search`：遞迴搜尋（RLM 風格，BM25+Vector+LLM Reranker）
- `/kb-new`：寫新卡片（搜 KB 建 wiki-link，append-only 原則）
- `/kb-digest`：增量精煉（IaC 交叉驗證、wiki-link、矛盾處理、不確定性保留）
- `/kb-publish`：合併上傳 NotebookLM（diff-based、敏感掃描）
- `/kb-verify`：品質驗證（敏感資訊、frontmatter、內容品質抽查）
- `/kb-feedback`：知識缺口分析（5 維度自動偵測）

## 「Redis 之前出過什麼問題？怎麼解決的？」

每小時 cache refresh 時約 5% request error（Redis timeout）。

根因：G1GC Stop-The-World pause 凍結 JVM 所有 thread（包含 Netty EventLoop），Redis server 已回應但 client 端無法處理。

解決：
1. 改 shareNativeConnection=false，分散 connection → 錯誤從 5% 降到 <1%
2. Application-level warmup（Spring Boot 啟動前預熱）
3. JVM 參數調整（IHOP 45→35）

## 「這個系統如何確保不會洩漏敏感資訊？」

兩道閘門，每道都是 blocker（不是 warning）：
1. `/kb-digest` 產出時掃描 → 有敏感資訊就阻塞 commit
2. `/kb-publish` 上傳前掃描 → 有敏感資訊就阻塞上傳

掃描：密碼、API key、token、AWS key、IP 位址等。

## 「知識還在 Jira/Confluence 裡，要全部搬過來嗎？」

不用遷移。遇到再寫就好：
- 排障時翻到 Jira 上的解法 → 順手 `/kb-new` 記一張卡片
- 沒人再查的東西，就讓它留在原處
- `/kb-feedback` 會自動偵測缺口（有 IaC module 但沒文件），這時才去補

知識庫不需要「完整」才有用。22 篇精煉文件已經能回答問題了，每多一張卡片價值就多一點。

## 「為什麼不統一寫 Confluence 就好？」

問題不是寫在哪裡，是寫了之後誰來處理。

- **輸入端**：Confluence 要排版、分類、放對位置——門檻高所以沒人寫。KB 零摩擦，隨手記 markdown 就好。
- **輸出端**：Confluence 寫了就放著，沒人更新。KB 有 AI 精煉、建連結、標過時、語意搜尋。

Confluence 是更好的抽屜，這個 KB 是會自己整理的抽屜。
