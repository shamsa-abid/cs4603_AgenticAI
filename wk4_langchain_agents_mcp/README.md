# Week 4: LangChain Agents — Multi-Agents, MCP & Middleware

This week builds on the agent fundamentals from Week 3 and covers three advanced topics: **multi-agent coordination**, the **Model Context Protocol (MCP)**, and **agent middleware** (message management + human-in-the-loop).

---

## 📁 1. Multi-Agents (`1.multiagents/`)

### `1.agent-sql.ipynb` — SQL Agents
Builds a LangChain agent that queries a SQLite database (Chinook music catalog) using a custom `@tool`. Demonstrates the difference between a naive agent and one augmented with schema hints, and shows how to dynamically inject the database schema into the system prompt for more accurate SQL generation.

**Key APIs:** `SQLDatabase.from_uri`, `@tool`, `create_agent`, `SystemMessage`, runtime schema extraction

---

### `2.langchain_multi_agent.ipynb` — Agent-as-Tool Pattern
Shows how to wrap a specialised subagent (e.g., a math agent) as a callable `@tool` so a higher-level orchestrator agent can delegate to it. Uses `square_root` and `square` math agents as examples of the modular "agent-as-tool" composition pattern.

**Key APIs:** `@tool`, `create_agent`, subagent wrapping, tool delegation

---

### `3.multi-agent-eventplanner.ipynb` — Coordinated Multi-Agent System
Coordinates multiple specialised agents (venue search + music recommendation) to collaboratively plan an event. Demonstrates shared `AgentState`, inter-agent communication via `Command`, and persistent conversation memory with checkpointing and thread IDs.

**Key APIs:** `AgentState`, `ToolRuntime`, `Command`, `MemorySaver`, `create_agent`, `@tool`

---

## 📁 2. Model Context Protocol (`2.mcp/`)

### `4.1.mcp_server_basic.py` — Basic MCP Server
A minimal FastMCP server that exposes a **web search tool**, a **GitHub README resource**, and a **prompt template** for LangChain/LangGraph questions. Supports both `stdio` and HTTP (streamable) transports.

**Key APIs:** `FastMCP`, `@mcp.tool`, `@mcp.resource`, `@mcp.prompt`, `uvicorn`

---

### `4.2.mcp_client.ipynb` — MCP Client with LangChain
Connects to a running MCP server, discovers its tools, inspects their schemas, and invokes them from within a LangChain workflow. Shows how MCP tools are consumed as standard LangChain-compatible tools and how to load MCP resources and prompts.

**Key APIs:** `MultiServerMCPClient`, `get_tools()`, `load_mcp_resources`, `load_mcp_prompt`

---

### `4.4.mcp_server_db.py` — Database-Backed MCP Server
An MCP server backed by the Chinook SQLite database. Exposes a safe parameterised SQL search tool, a schema resource, and a music analyst prompt. Also includes an LLM-augmented tool that summarises query results without hallucinating by grounding responses solely in retrieved rows.

**Key APIs:** `FastMCP`, `@mcp.tool`, `@mcp.resource`, `@mcp.prompt`, `sqlite3`, lazy LLM init, `uvicorn`

---

### `4.5.mcp_client_db.py` — CLI Client for DB MCP Server
A command-line Python client that launches the Chinook MCP server as a child process over `stdio`, then provides a menu to search tracks, render prompts, or read server resources. Demonstrates low-level MCP client usage outside of LangChain.

**Key APIs:** `ClientSession`, `StdioServerParameters`, `stdio_client`, `call_tool`, `get_prompt`, `read_resource`

---

## 📁 3. Middleware (`3.middleware/`)

### `0.middleware_lifecycle.ipynb` — The Middleware Lifecycle
Visualises the exact order in which middleware hooks fire during an agent loop. Attaches a logger to every hook (`before_agent`, `before_model`, `wrap_model_call`, `after_model`, `wrap_tool_call`, `after_agent`) so you can watch the real execution order.

**Key APIs:** `before_agent`, `before_model`, `after_model`, `after_agent`, `wrap_model_call`, `wrap_tool_call`

---

### `1.managing_messages.ipynb` — Message Management
Covers strategies for controlling what gets passed to the LLM — trimming long histories, filtering by role or count, and removing specific messages. Shows how to build custom middleware that intercepts the agent's message list before each model call.

**Key APIs:** `trim_messages`, `filter_messages`, `RemoveMessage`, `AgentState`, `before_agent`

---

### `2.human_inthe_loop.ipynb` — Human-in-the-Loop Approval
Builds an email assistant where reading email runs automatically but **sending email requires human approval**. The agent pauses mid-execution, the human can approve, edit, or reject, and execution resumes on the same thread via `Command`.

**Key APIs:** `HumanInTheLoopMiddleware`, `AgentState`, `ToolRuntime`, `Command`, checkpointers, interrupt/resume

---

### `3.hitl_refund.py` — Human-Approved Refund Workflow
A command-line refund assistant that looks up an order automatically, then **pauses before issuing a refund** for human review. Demonstrates a practical HITL pattern using a checkpointed agent thread that can be safely paused and resumed.

**Key APIs:** `create_agent`, `HumanInTheLoopMiddleware`, `InMemorySaver`, `Command(resume=...)`, `@tool`

---

### `4.langchain_guardrails.ipynb` — Guardrails
Demonstrates safety checks that wrap an LLM agent: PII redaction/masking, input guardrails (blocking off-topic requests), output guardrails (filtering unsafe answers), and usage limits (model/tool call caps to prevent runaway cost or infinite loops).

**Key APIs:** `PIIMiddleware`, `@before_model`, `@after_model`, `ModelCallLimitMiddleware`, `ToolCallLimitMiddleware`

---

### `5.dynamic_prompts.ipynb` — Dynamic Prompts
Shows how to build the system prompt **at runtime** instead of hard-coding it. A `@dynamic_prompt` middleware reads typed runtime context (e.g., customer tier) and returns a tailored system prompt for each model call — enabling personalisation, localisation, and persona switching.

**Key APIs:** `@dynamic_prompt`, `ModelRequest`, `context_schema`, runtime context dataclasses

---

### `6.retries_and_fallback.ipynb` — Reliability: Retries & Fallback
Makes agents resilient to transient failures. `ToolRetryMiddleware` re-runs failing tools with backoff, `ModelRetryMiddleware` retries flaky model calls, and `ModelFallbackMiddleware` switches to a backup model when the primary is unavailable.

**Key APIs:** `ToolRetryMiddleware`, `ModelRetryMiddleware`, `ModelFallbackMiddleware`

---

### `7.tool_selection.ipynb` — Scaling Tools: LLM Tool Selector
When an agent has many tools, `LLMToolSelectorMiddleware` uses a lightweight LLM step to pick only the **most relevant tools** for the current question before the main model runs — reducing cost, latency, and selection errors.

**Key APIs:** `LLMToolSelectorMiddleware`, `max_tools`

---

### `8.custom_wrap_hooks.ipynb` — Custom `wrap_model_call` & `wrap_tool_call`
The most powerful middleware pattern: intercept model or tool calls to time, log, cache, rewrite, or **short-circuit** them. Demonstrates timing/token logging, appending a signature to replies, and blocking high-value refunds at the tool level.

**Key APIs:** `@wrap_model_call`, `@wrap_tool_call`, `handler(request)`, `ToolMessage`

---

### `9.context_editing.ipynb` — Automatic Context Editing
Long-running agents accumulate bulky tool outputs. `ContextEditingMiddleware` automatically prunes old tool results once the conversation crosses a token threshold — keeping the most recent ones and replacing older blobs with a placeholder to save tokens.

**Key APIs:** `ContextEditingMiddleware`, `ClearToolUsesEdit`, `trigger`, `keep`, `placeholder`

---

## Topic Map

| Area | Files |
|------|-------|
| SQL Agents | `1.agent-sql.ipynb` |
| Multi-Agent Composition | `2.langchain_multi_agent.ipynb`, `3.multi-agent-eventplanner.ipynb` |
| MCP Server | `4.1.mcp_server_basic.py`, `4.4.mcp_server_db.py` |
| MCP Client | `4.2.mcp_client.ipynb`, `4.5.mcp_client_db.py` |
| Middleware Lifecycle | `0.middleware_lifecycle.ipynb` |
| Message Middleware | `1.managing_messages.ipynb` |
| Human-in-the-Loop | `2.human_inthe_loop.ipynb`, `3.hitl_refund.py` |
| Guardrails | `4.langchain_guardrails.ipynb` |
| Dynamic Prompts | `5.dynamic_prompts.ipynb` |
| Retries & Fallback | `6.retries_and_fallback.ipynb` |
| Tool Selection | `7.tool_selection.ipynb` |
| Custom Wrap Hooks | `8.custom_wrap_hooks.ipynb` |
| Context Editing | `9.context_editing.ipynb` |
