---
name: ai-agents
description: "AI agents, MCP servers, LLM integrations, RAG, tool use. Use for building AI features. Do NOT use for code review, deployment, or non-AI application logic."
---

# AI & Agent Building

Build AI-powered features, autonomous agents, MCP servers, and LLM integrations. From chat features to full agentic systems.

## AI Integration Options

| Approach | Best For |
|----------|----------|
| **Vercel AI SDK** | Web apps (Next.js/React) — streaming, hooks, multi-provider |
| **Anthropic SDK** | Backend services, custom integrations |
| **tRPC + AI** | Type-safe AI endpoints in full-stack TypeScript |

## Key Patterns

### Streaming (Always for User-Facing)
Always stream AI responses for user-facing features. Use `streamText()` (Vercel AI SDK) or `client.messages.stream()` (Anthropic SDK).

### Tool Use / Function Calling
Let the AI call your functions. Define tools with typed schemas (Zod), handle `tool_use` blocks in responses, send results back for final response.

### RAG (Retrieval-Augmented Generation)
1. Embed documents → store in vector DB (Pinecone, pgvector)
2. At query time: embed question → find similar docs → feed context to LLM

## Building MCP Servers

Model Context Protocol servers expose tools to Claude Code and Claude Desktop.

**TypeScript**: Use `@modelcontextprotocol/sdk` with `McpServer` + `StdioServerTransport`
**Python**: Use `mcp.server.fastmcp` with `FastMCP`

### MCP Tool Design Principles
- **One tool, one job** — no Swiss Army knife tools
- **Descriptive names** — `search_users` not `su`
- **Rich descriptions** — explain what, when, and returns
- **Typed inputs** — Zod schemas with field descriptions
- **Graceful errors** — return error messages, don't throw
- **Pagination** — cursor-based for list endpoints

## Building Agents

### Architecture Patterns

**ReAct Loop** (Recommended): Observe → Think → Act → repeat until done. Use tool_use responses to execute functions, feed results back.

**Orchestrator-Worker**: One agent plans, spawns specialized worker agents in parallel or sequence.

**Pipeline**: Sequential processing with handoffs: Input → Extract → Transform → Validate → Output.

## Prompt Engineering

### System Prompt Structure
```
[Role] → [Context] → [Task] → [Constraints] → [Output Format] → [Examples]
```

### Key Techniques
- Be specific: "Respond in 2-3 sentences" not "Be concise"
- Use examples: Show input/output pairs
- Chain of thought: "Think step by step"
- Structured output: Ask for JSON/XML with specific structure
- Negative examples: "Do NOT include disclaimers"

## Model Selection

| Use Case | Model |
|----------|-------|
| Complex reasoning | Claude Opus |
| General tasks | Claude Sonnet |
| High-volume, simple | Claude Haiku |
| Embeddings | text-embedding-3-small |

## Reference Docs

Read `references/mcp-patterns.md` for MCP server design patterns and examples.
Read `references/plugin-building.md` for Claude Code plugin/skill creation guide.
Read `references/ai-integration-code.md` for Vercel AI SDK, Anthropic SDK, tool use, streaming, and RAG code examples.
