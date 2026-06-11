# Java AI 多模块项目 — 架构与流程图

> 基于典型 Java AI 课程代码仓库梳理，供 OpenSpec / 开发者理解多模块结构。  
> 项目背景可写入 [`openspec/config.yaml`](../openspec/config.yaml)。

---

## 1. 仓库总览

```mermaid
flowchart TB
    subgraph ROOT["多模块仓库（无根 pom，各模块独立）"]
        direction TB

        subgraph TUTORIAL["📚 基础教程"]
            SA["spring-ai :8082"]
            SAA["spring-ai-alibaba :8083"]
            PR["prompt :8084"]
            LC["langchain4j-ai :8085"]
            AA["agent-ai :8086"]
            AS["agent-scope-ai :8080"]
            EF["eval-framework"]
        end

        subgraph PROTOCOL["🔗 协议演示（成对运行）"]
            MCP_S["mcp-tools-server :8090"]
            MCP_C["mcp-tools-client"]
            A2A_S["a2a-server :8080"]
            A2A_C["a2a-client :8090"]
        end

        subgraph PRODUCT["🏗️ 完整实战项目"]
            RAG["jc-rag-kb :8080"]
            RAG_F["jc-rag-kb-front"]
            SALES["jc-sales-agent :8087"]
            SALES_F["jc-sales-agent-front"]
            VOICE["jc-voice-shopping :8080"]
        end

        subgraph TOOLING["📄 工具链"]
            DOC["docs/"]
            OS["openspec/"]
        end
    end

    RAG_F -->|HTTP proxy| RAG
    SALES_F -->|HTTP proxy| SALES
    MCP_C -->|SSE/HTTP| MCP_S
    A2A_C -->|A2A Protocol| A2A_S
```

---

## 2. 子项目独立性与配对关系

```mermaid
graph LR
    subgraph 完全独立["Maven 完全独立（无 cross-module 依赖）"]
        A1[spring-ai]
        A2[spring-ai-alibaba]
        A3[prompt]
        A4[langchain4j-ai]
        A5[agent-ai]
        A6[agent-scope-ai]
        A7[jc-voice-shopping]
        A8[eval-framework]
    end

    subgraph 运行时配对["运行时配对（需同时启动）"]
        B1[mcp-tools-server] --- B2[mcp-tools-client]
        C1[a2a-server] --- C2[a2a-client]
        D1[jc-rag-kb] --- D2[jc-rag-kb-front]
        E1[jc-sales-agent] --- E2[jc-sales-agent-front]
    end

    subgraph 共享基础设施["共享外部 DB（配置级）"]
        F1[(MySQL 共享库)]
        F2[(PostgreSQL)]
        F3[(Redis)]
        F4[(MinIO)]
    end

    A1 & A3 & A5 & A4 --> F1
    D1 --> F2 & F3 & F4
    A7 --> F2 & F3
```

---

## 3. spring-ai-alibaba 教程内部分层

```mermaid
flowchart LR
    subgraph Controllers["controller 包（按主题）"]
        CHAT[chat<br/>基础对话]
        VEC[vector / rag / know<br/>向量与RAG]
        MULTI[multi / parallel<br/>多模型与并行]
        MODAL[vision / imageGenerate / voice<br/>多模态]
        ERR[error<br/>重试与容错]
    end

    subgraph Services["service 层"]
        SVC[Chat / RAG / Vector / Parallel Services]
    end

    subgraph External["外部依赖"]
        DS[DashScope API]
        PG[(PostgreSQL pgvector)]
    end

    CHAT & VEC & MULTI & MODAL & ERR --> SVC
    SVC --> DS & PG
```

---

## 4. jc-rag-kb — RAG 查询时序图

```mermaid
sequenceDiagram
    actor User as 用户/前端
    participant Front as jc-rag-kb-front
    participant API as jc-rag-kb
    participant Auth as AuthController
    participant RAG as RagQueryController
    participant Embed as EmbeddingModel
    participant PG as PostgreSQL+pgvector
    participant Rerank as Reranker API
    participant LLM as ChatModel (Qwen)

    User->>Front: 输入问题
    Front->>API: POST /api/rag/query
    API->>Auth: 校验 Token（如需要）
    Auth-->>API: OK

    API->>Embed: 问题向量化
    Embed-->>API: query vector

    API->>PG: 向量相似度检索 Top-K
    PG-->>API: 候选文档 chunks

    API->>Rerank: 精排（gte-rerank-v2）
    Rerank-->>API: Top-N 片段

    API->>LLM: Prompt = 上下文 + 问题
    LLM-->>API: 流式/完整回答

    API-->>Front: 回答 + 引用来源
    Front-->>User: 展示 Markdown 回答
```

---

## 5. MCP — Server/Client 调用时序图

```mermaid
sequenceDiagram
    participant Client as mcp-tools-client
    participant Transport as HTTP/SSE Transport
    participant Server as mcp-tools-server :8090
    participant Tools as BusinessTools
    participant LLM as ChatModel (DeepSeek)

    Client->>Transport: 连接 localhost:8090
    Transport->>Server: SSE 握手
    Server-->>Client: 工具列表 (list_tools)

    Client->>LLM: 用户问题 + MCP 工具描述
    LLM-->>Client: tool_call 请求

    Client->>Server: call_tool(name, args)
    Server->>Tools: 执行 BusinessTools 方法
    Tools-->>Server: 业务结果
    Server-->>Client: tool 结果

    Client->>LLM: 附带 tool 结果继续对话
    LLM-->>Client: 最终回答
```

---

## 6. A2A — Agent 协作时序图

```mermaid
sequenceDiagram
    participant Client as a2a-client :8090
    participant Registry as AgentRegistrar
    participant Server as a2a-server :8080
    participant Agent as Remote Agent
    participant Webhook as Client Webhook

    Client->>Registry: register(localhost:8080)
    Registry->>Server: 获取 Agent Card
    Server-->>Registry: Agent 元数据

    Client->>Server: 发起 A2A 任务（streaming）
    Server->>Agent: 执行任务
    Agent-->>Server: 进度/中间结果
    Server-->>Client: SSE 流式推送

    opt 长时任务
        Server->>Webhook: POST /webhook/a2a-callback
        Webhook-->>Client: 异步通知完成
    end

    Server-->>Client: 任务完成结果
```

---

## 7. jc-voice-shopping — 语音导购流程图

```mermaid
flowchart TD
    START([用户语音输入]) --> WS[WebSocket 音频流]
    WS --> ASR[AsrService 语音识别]
    ASR --> INTENT[IntentService 意图识别]

    INTENT -->|闲聊| CHITCHAT[ChitchatTemplates]
    INTENT -->|商品咨询| REC[RecommendOrchestrator]
    INTENT -->|下单| ORDER[OrderService]
    INTENT -->|需澄清| CLARIFY[ClarifyService]

    REC --> PV[ProductVectorService 向量检索]
    REC --> FAQ[FaqVectorService]
    REC --> PROFILE[UserProfileService 画像重排]
    PV & FAQ & PROFILE --> RERANK[ProfileReranker]
    RERANK --> TTS[TtsService 语音合成]

    ORDER --> CONFIRM[CommonConfirmer 确认]
    CONFIRM --> PENDING[PendingOrderStore]
    PENDING --> TTS

    CHITCHAT --> TTS
    CLARIFY --> TTS

    TTS --> WS_OUT[WebSocket 音频输出]
    WS_OUT --> END([用户听到回复])

    subgraph 记忆与状态
        SESSION[SessionStateService]
        MEM[LongTermMemoryWriter]
    end

    INTENT --> SESSION
    REC --> MEM
```

---

## 8. jc-sales-agent — 销售分析 Agent 时序图

```mermaid
sequenceDiagram
    actor User as 用户
    participant Front as jc-sales-agent-front
    participant API as jc-sales-agent :8087
    participant Agent as SalesAgentController
    participant Tools as SQL/Chart Tools
    participant DB as MySQL
    participant LLM as ChatModel

    User->>Front: 自然语言提问
    Front->>API: POST /agent/chat (SSE)
    API->>Agent: 解析请求

    loop Agent ReAct 循环
        Agent->>LLM: 思考 + 可选 tool_call
        LLM-->>Agent: 调用 SQL 查询工具
        Agent->>Tools: 执行 SQL
        Tools->>DB: SELECT ...
        DB-->>Tools: 数据行
        Tools-->>Agent: 查询结果
        Agent->>LLM: 附带数据继续推理
    end

    LLM-->>Agent: 分析报告（含图表描述）
    Agent-->>Front: SSE 流式输出
    Front-->>User: Markdown + ECharts 渲染
```

---

## 9. 端口速查表

| 端口 | 子项目 | 备注 |
|------|--------|------|
| 8080 | jc-rag-kb / jc-voice-shopping / agent-scope-ai / a2a-server | ⚠️ 冲突，不可同时启动 |
| 8082 | spring-ai | |
| 8083 | spring-ai-alibaba | |
| 8084 | prompt | |
| 8085 | langchain4j-ai | |
| 8086 | agent-ai | |
| 8087 | jc-sales-agent | |
| 8090 | mcp-tools-server / a2a-client | ⚠️ 冲突 |
| 5173 | 两个前端 dev server | Vite 默认 |

---

## 10. 维护说明

- 新增子项目后，同步更新 `openspec/config.yaml` 与本文件
- 远程 DB 地址以各模块 `application.yml` 为准
- 架构变更可通过 OpenSpec：`/opsx-propose` → `/opsx-apply` → `/opsx-archive`
