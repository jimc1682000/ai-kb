# AI Knowledge Base

> https://github.com/jimc1682000/ai-kb

用 AI 驅動的知識庫系統，結合卡片盒筆記法（Zettelkasten）與 Claude Code，實現團隊知識的原子化、互連與漸進累積。

## 簡介

- 解決找資料翻多個工具、同一個坑反覆踩、寫文件門檻高的痛點
- 以 Obsidian 為編輯介面，Claude Code Skills 為 AI 後端
- 支援個人 KB 與團隊 KB 的雙層架構

## 主要功能

| Skill | 說明 |
|-------|------|
| `/kb-new` | 建立新知識卡片 |
| `/kb-search` | 語意搜尋知識庫 |
| `/kb-verify` | 驗證卡片品質 |
| `/kb-publish` | 發布到團隊 KB |
| `/kb-feedback` | 回饋與改善 |
| `/kb-digest` | 產生知識摘要 |

## 文件結構

```
index.md              # 簡報首頁
motivation.md         # 動機與痛點
zettelkasten.md       # 卡片盒筆記法
architecture.md       # 系統架構
ai-agents.md          # AI Agent 設計
ai-skills.md          # Skills 總覽
skill-kb-*.md         # 各 Skill 詳細說明
security.md           # 安全機制
metrics.md            # 成效指標
takeaway.md           # 重點摘要
_presenter/           # 講者備忘
_nlm-sources/         # NotebookLM 來源
```

## License

[MIT](./LICENSE)
