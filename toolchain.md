# 工具鏈

> 附錄：這套系統使用到的所有工具清單。

## 核心工具

| 工具                                                                 | 用途                                     | 類型          |
| ------------------------------------------------------------------ | -------------------------------------- | ----------- |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code)      | AI coding agent，執行 skill 的 runtime     | AI Agent    |
| [npx skills](https://github.com/vercel-labs/skills)                | Skill 套件管理（安裝/更新/列出）                   | CLI         |
| [Obsidian](https://obsidian.md/)                                   | 知識庫閱覽器（vault + graph view + wiki-link） | Desktop App |
| [NotebookLM](https://notebooklm.google.com/)                       | 語意問答 + Deep Research + [[notebooklm-integration|多元輸出]]  | Web App     |
| [nlm CLI](https://github.com/jacob-bd/notebooklm-mcp-cli) | NotebookLM API 操作                      | CLI / MCP   |
| Git                                                                | 版本控制（所有 repo）                          | CLI         |

## 開發 / 輔助工具

| 工具 | 用途 |
|------|------|
| Gitea | 自建 Git server + code review（PR） |
| Mermaid | 架構圖繪製 |
| jq | JSON 處理（NLM CLI 輸出解碼） |

## MCP（Model Context Protocol）

nlm CLI 支援 MCP 整合，讓 Claude Code 可以直接操作 NotebookLM：

```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "nlm",
      "args": ["mcp"]
    }
  }
}
```

這代表 AI 可以在 skill 中直接：
- 列出 notebook 的 sources
- 上傳新 source
- 查詢 notebook
- 刪除舊 source

## Skill 的工具權限

每個 skill 在 `SKILL.md` 的 frontmatter 中宣告可用工具：

```yaml
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
```

這是安全邊界——skill 只能使用被允許的工具。

→ 回到 [[index]]
→ 了解安全設計 [[security]]
