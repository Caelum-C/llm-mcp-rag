# 项目经历

## LLM-MCP-RAG 智能代理系统

**项目背景**

随着大语言模型（LLM）的普及，如何让 LLM 超越自身知识局限、与外部世界交互成为核心诉求。本项目基于 Golang 实现了一个轻量级智能代理（Agent）Demo，通过集成 OpenAI GPT、MCP（Model Context Protocol）协议和 RAG（检索增强生成）模式，使 LLM 能够自主规划并调用外部工具（如网页抓取、文件读写）来完成复杂任务，验证了"LLM + 工具调用"的 Agent 架构可行性。

**技术栈**

- **编程语言**：Go 1.24
- **LLM 接入**：OpenAI API（`openai-go/v3`），支持 GPT-3.5 / GPT-4 系列模型，基于 Streaming 流式响应
- **工具协议**：MCP（Model Context Protocol），通过 `mark3labs/mcp-go` 客户端库，采用 stdio JSON-RPC 传输与 MCP Server 通信
- **外部工具**：`mcp-server-fetch`（网页内容获取）、`@modelcontextprotocol/server-filesystem`（本地文件读写）
- **架构模式**：Agent Loop（工具调用循环）、RAG 上下文注入、多轮对话消息管理

**担任角色**

独立开发者，负责系统架构设计与全部核心模块的编码实现。

**主要工作与成果**

- 设计并实现了三层解耦架构：`MCPClient`（工具协议层）→ `ChatOpenAI`（LLM 交互层）→ `Agent`（编排调度层），各层职责清晰，可独立扩展。
- 实现了 MCP 协议客户端，支持在运行时动态发现并注册 MCP Server 提供的工具列表，将 MCP 工具定义自动转换为 OpenAI Function Calling 格式，无需手动维护工具 Schema。
- 实现了基于流式响应的 Agent Loop：LLM 返回工具调用指令 → 路由至对应 MCP Client 执行 → 将结果回填对话上下文 → 再次驱动 LLM 推理，直至任务完成，支持多轮工具链式调用。
- 支持通过 RAG 上下文（`ragCtx`）向 LLM 注入领域知识或业务约束，提升输出质量与可控性。
- 以"访问 Hacker News 首页并将摘要写入本地 Markdown 文件"为端到端验证场景，完整走通了"网页获取 → 内容摘要 → 文件写入"的自动化工作流。

**解决的核心问题**

- **工具异构问题**：通过统一的 MCP 协议屏蔽了不同工具服务的实现差异，新增工具只需接入 MCP Server，无需修改 Agent 核心逻辑。
- **流式响应解析问题**：OpenAI Streaming 模式下工具调用信息分片返回，实现了分片累积与结构化解析，确保工具调用参数完整提取。
- **多工具路由问题**：Agent 同时管理多个 MCP Client，通过工具名称精确匹配将 LLM 的调用请求路由到正确的客户端，避免工具命名冲突。
