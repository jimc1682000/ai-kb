# /kb-verify — 品質驗證

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

→ 安全設計 [[security]]
→ 回到 [[ai-skills]]
