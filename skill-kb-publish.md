# /kb-publish — 上傳 NotebookLM

把精煉文件合併後上傳到 Google NotebookLM。

- 按 `categories` 合併（每個 category 一個 source）
- 上傳前敏感掃描（阻塞門檻）
- **Diff-based**：只重新上傳有變更的 category
- NLM 不支援「更新」→ 刪除舊 source + 重新上傳
- 支援 test / prod 環境選擇
- **元資料保留**：合併時保留日期和確定性等級，NLM 回答可標註時效性和可信度

→ NotebookLM 整合 [[notebooklm-integration]]
→ 安全設計 [[security]]
→ 回到 [[ai-skills]]
