# /kb-search — 遞迴搜尋

知識庫的查詢入口。借鑑 RLM（Recursive Language Models）概念，LLM 自主判斷相關性、決定往哪裡深入。

- **Progressive Collection Search**：先搜 digest 取得結構化概覽 → 再深入 raw notes / Library / IaC
- **qmd 混合引擎**：BM25 關鍵字 + Vector 語意 + LLM Reranker 三層融合
- **Query 意圖分類**：
  - 聚焦 query（如「redis timeout」）→ 直接搜
  - 廣泛 query（如「jenkins」）→ 先搜，列出主題方向讓使用者選
  - 模糊 query（如「deployment」）→ 先問使用者意圖再搜
- **LLM 遞迴決策**：每輪搜尋後 LLM 決定下一步——搜 raw、查 IaC、展開 wiki-link、還是詢問使用者
- **--quick 模式**：BM25 全掃，秒級回答

> 💡 **專案啟動建議**：新專案開始前，用 `/kb-search` 搜尋相關知識，再手動把需要的檔案複製到專案 `docs/` 作為 reference。

→ 搜尋技術背景 [[search-concepts]]
→ 回到 [[ai-skills]]
