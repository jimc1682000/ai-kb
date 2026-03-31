# /kb-new — 寫一張新卡片

遵循[[zettelkasten|卡片盒筆記法]]的 append-only 原則——永遠寫新卡片，不修改既有卡片。

```
使用者：「記一下 Redis timeout 的問題」
AI：讀上下文 → 搜 KB 找相關卡片 → 判斷模板 → 問答補齊 → 寫入 + 建 link → git commit & push
```

- **搜 KB 建 link**：寫之前先搜相關卡片，自動建立 wiki-link（續卡、修正、分支）
- AI 自動判斷模板類型（troubleshooting / architecture / howto / reference / general）
- 產生語義化檔名（`redis-connection-timeout.md`）
- 缺少資訊時一次問完，不逐一問
- 寫入後自動 commit + push

→ 卡片盒原則 [[zettelkasten]]
→ 回到 [[ai-skills]]
