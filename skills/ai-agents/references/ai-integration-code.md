# AI Integration Code Examples

## Vercel AI SDK (Web Apps)

### API Route
```typescript
// app/api/chat/route.ts
import { anthropic } from "@ai-sdk/anthropic";
import { streamText } from "ai";

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-20250514"),
    system: "You are a helpful assistant.",
    messages,
  });

  return result.toDataStreamResponse();
}
```

### React Chat Component
```tsx
// components/Chat.tsx
"use client";
import { useChat } from "ai/react";

export function Chat() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat();

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id} className={m.role === "user" ? "text-right" : "text-left"}>
          {m.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} placeholder="Ask something..." />
        <button type="submit" disabled={isLoading}>Send</button>
      </form>
    </div>
  );
}
```

## Anthropic SDK (Direct API)

### Basic Usage
```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic(); // uses ANTHROPIC_API_KEY env var

const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Explain quantum computing simply." }],
});
```

### Tool Use / Function Calling
```typescript
const result = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  tools: [
    {
      name: "get_weather",
      description: "Get current weather for a location",
      input_schema: {
        type: "object",
        properties: {
          location: { type: "string", description: "City name" },
        },
        required: ["location"],
      },
    },
  ],
  messages: [{ role: "user", content: "What's the weather in London?" }],
});

// Handle tool use in the response
for (const block of result.content) {
  if (block.type === "tool_use") {
    const toolResult = await executeFunction(block.name, block.input);
    // Send tool result back for final response
  }
}
```

### Streaming
```typescript
const stream = await client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [{ role: "user", content: prompt }],
});

for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

## MCP Server (TypeScript)
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool(
  "search_docs",
  "Search documentation for a query",
  { query: z.string().describe("Search query") },
  async ({ query }) => {
    const results = await searchDocs(query);
    return { content: [{ type: "text", text: JSON.stringify(results) }] };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## MCP Server (Python)
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def search_docs(query: str) -> str:
    """Search documentation for a query."""
    results = do_search(query)
    return json.dumps(results)

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

## ReAct Agent Loop
```typescript
async function agentLoop(task: string, tools: Tool[], maxSteps = 10) {
  const messages = [{ role: "user", content: task }];

  for (let step = 0; step < maxSteps; step++) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4096,
      tools: tools.map(t => t.definition),
      messages,
    });

    messages.push({ role: "assistant", content: response.content });

    if (response.stop_reason === "end_turn") break;

    const toolResults = [];
    for (const block of response.content) {
      if (block.type === "tool_use") {
        const result = await tools.find(t => t.name === block.name)?.execute(block.input);
        toolResults.push({ type: "tool_result", tool_use_id: block.id, content: result });
      }
    }
    messages.push({ role: "user", content: toolResults });
  }

  return messages;
}
```

## RAG Pipeline
```typescript
// 1. Embed documents
const embedding = await openai.embeddings.create({
  model: "text-embedding-3-small",
  input: documentText,
});

// 2. Store in vector database
await vectorDB.upsert({
  id: docId,
  values: embedding.data[0].embedding,
  metadata: { text: documentText },
});

// 3. At query time: embed the question, find similar docs
const queryEmbedding = await openai.embeddings.create({
  model: "text-embedding-3-small",
  input: question,
});
const results = await vectorDB.query({
  vector: queryEmbedding.data[0].embedding,
  topK: 5,
});

// 4. Feed context to LLM
const answer = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  messages: [{
    role: "user",
    content: `Context:\n${results.map(r => r.metadata.text).join("\n\n")}\n\nQuestion: ${question}`
  }],
});
```
