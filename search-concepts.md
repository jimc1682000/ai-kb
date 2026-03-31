# 搜尋技術：RAG → Agentic RAG → RLM

## 演化光譜

```mermaid
flowchart LR
    N["1 Naive RAG<br/>固定 pipeline<br/>搜一次定生死"]
    S["2 Self-RAG<br/>模型決定<br/>要不要搜"]
    A["3 Active Retrieval<br/>模型決定<br/>何時搜"]
    AG["4 Agentic RAG<br/>模型使用<br/>搜尋工具"]
    R["5 RLM<br/>遞迴分解<br/>自主探索"]

    N --> S --> A --> AG --> R

    style N fill:#c8e6c9
    style S fill:#a5d6a7
    style A fill:#81d4fa
    style AG fill:#64b5f6
    style R fill:#ffcc80

    linkStyle 0,1,2,3 stroke:#999,stroke-width:2px
```

```
  系統決定 ◄──────────────────────────────────► 模型決定
  被動接收結果                                      主動設計搜尋策略

                          ▲
                      我們在這裡
                     （層級 3-4）
```

## 三個名詞一張圖

```mermaid
flowchart LR
    subgraph rag ["RAG"]
        A["使用者問題"] --> B["向量搜尋<br/>找出相關 chunk"]
        B --> C["LLM 根據 chunk 回答"]
    end

    subgraph agentic ["Agentic RAG"]
        D["使用者問題"] --> E["LLM 判斷<br/>要不要搜？搜什麼？"]
        E --> F["搜尋"]
        F --> G{"結果夠好？"}
        G -->|不夠| H["改寫 query<br/>換資料來源"]
        H --> F
        G -->|夠了| I["LLM 整合回答"]
    end

    subgraph rlm ["RLM"]
        J["使用者問題"] --> K["Context 存為<br/>REPL 變數"]
        K --> L["LLM 寫 Python<br/>探索、分解變數"]
        L --> M{"子問題？"}
        M -->|是| O["遞迴 sub-call<br/>自己處理子問題"]
        O --> L
        M -->|否| N["整合回答"]
    end

    style rag fill:#e8f5e9
    style agentic fill:#e3f2fd
    style rlm fill:#fff3e0
```

## 白話解釋

### RAG（Retrieval-Augmented Generation）

「先找再答」——把使用者的問題拿去搜資料庫，找到相關段落，再讓 LLM 根據這些段落回答。

- **優點**：快、穩定、架構簡單
- **弱點**：chunk 斷章取意（300 token 的片段失去前後文）、搜一次就定生死

### Agentic RAG

「找不好就再找」——LLM 不只被動接收搜尋結果，還能主動：
- 決定要不要搜（有些問題不需要 retrieval）
- 評估結果品質（不好就改寫 query 重搜）
- 切換資料來源（這邊找不到，換個地方找）
- 多步推理（先搜 A 的結果用來搜 B）

### RLM（Recursive Language Models）

「模型自己寫程式碼探索」——最激進的一端。把 context 存為 REPL 環境中的變數，LLM 寫任意 Python 程式碼來檢視、分解這個變數，遇到子問題就遞迴呼叫自己（`llm_query`）處理。

- **核心機制**：context → REPL 變數 → LLM 寫 code 探索 → 遞迴 sub-call
- **優點**：能處理超過 context window 100 倍的輸入、有完整上下文不跑偏
- **弱點**：compute 成本高、對小量文件殺雞用牛刀

## 我們的 /kb-search 在哪裡

```mermaid
flowchart TB
    subgraph our ["我們的 Hybrid 設計"]
        direction TB
        R1["Round 1【RAG 層】<br/>qmd 三層融合搜尋<br/>從 221 篇 → 3-5 篇候選<br/>秒級定位"]
        CP{"廣泛 query？"}
        ASK["列主題方向<br/>問使用者要深入哪個"]
        R2["Round 2+【RLM 層】<br/>讀完整文件 + 追溯 raw notes<br/>展開 wiki-link + 查 IaC<br/>LLM 自主決定下一步"]
        BACK["需要更多候選？<br/>→ 回 RAG 層 focused query"]

        R1 --> CP
        CP -->|是| ASK
        CP -->|否| R2
        ASK --> R2
        R2 --> BACK
        BACK -.-> R1
    end

    style our fill:#f3e5f5
```

**定位：光譜的層級 3-4 之間**

| | 純 RAG | 我們的 /kb-search | 純 RLM |
|---|---|---|---|
| 定位速度 | ✅ 秒級 | ✅ 秒級（RAG 層） | ❌ 逐篇探索 |
| 理解深度 | ❌ chunk 斷章取意 | ✅ 讀完整文件（RLM 層） | ✅ 完整上下文 |
| 搜尋策略 | 固定 pipeline | LLM 自主決定 | LLM 寫程式碼 |
| 跨文件追溯 | ❌ | ✅ wiki-link + sources | ✅ 遞迴呼叫 |

> **大顆粒用 RAG 加速定位，小顆粒用 RLM 精神深入理解。**

→ 回到 [[ai-skills]]
→ 回到 [[index]]
