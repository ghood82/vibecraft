# MCP Server Design Patterns

## Server Architecture

### stdio Server (Local)
Best for: Local tools, file system access, CLI wrappers
```
Claude ←→ stdio ←→ Your Server Process
```

### HTTP Server (Remote)
Best for: SaaS integrations, shared services, hosted APIs
```
Claude ←→ HTTP/SSE ←→ Your Server ←→ External API
```

## Tool Design Patterns

### CRUD Pattern
For resource management (users, documents, projects):
```typescript
server.tool("list_users", "List all users with optional filters", { ... });
server.tool("get_user", "Get a specific user by ID", { ... });
server.tool("create_user", "Create a new user", { ... });
server.tool("update_user", "Update an existing user", { ... });
server.tool("delete_user", "Delete a user by ID", { ... });
```

### Search + Detail Pattern
For discovery workflows:
```typescript
server.tool("search", "Search across all content", { query: z.string() });
server.tool("get_details", "Get full details of a specific item", { id: z.string() });
```

### Action Pattern
For imperative operations:
```typescript
server.tool("deploy", "Deploy the application to production", { environment: z.enum(["staging", "production"]) });
server.tool("rollback", "Rollback to previous deployment", { deployment_id: z.string() });
```

## Resource Design

Resources expose data that Claude can read:

```typescript
server.resource(
  "config",
  "application://config",
  "Current application configuration",
  async () => ({
    contents: [{ uri: "application://config", mimeType: "application/json", text: JSON.stringify(config) }],
  })
);
```

### Resource Templates (Dynamic)
```typescript
server.resource(
  "user-profile",
  new ResourceTemplate("users://{userId}/profile", { list: undefined }),
  "User profile data",
  async (uri, { userId }) => ({
    contents: [{ uri: uri.href, mimeType: "application/json", text: JSON.stringify(await getUser(userId)) }],
  })
);
```

## Error Handling

```typescript
server.tool("risky_operation", "...", schema, async (input) => {
  try {
    const result = await doSomething(input);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  } catch (error) {
    return {
      content: [{ type: "text", text: `Error: ${error.message}` }],
      isError: true,
    };
  }
});
```

## Authentication Patterns

### API Key (Simple)
```typescript
const apiKey = process.env.SERVICE_API_KEY;
if (!apiKey) throw new Error("SERVICE_API_KEY required");

async function callAPI(endpoint: string) {
  return fetch(`https://api.service.com${endpoint}`, {
    headers: { Authorization: `Bearer ${apiKey}` },
  });
}
```

### OAuth (For User-Scoped Access)
For MCP servers that need per-user auth, implement the OAuth flow and store tokens securely. The MCP SDK supports OAuth configuration in the transport layer.

## Testing MCP Servers

### Manual Testing
```bash
# Start server and send test messages
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | node server.js

# Use the MCP Inspector
npx @modelcontextprotocol/inspector node server.js
```

### Automated Testing
```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory.js";

test("search_docs returns results", async () => {
  const [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();
  await server.connect(serverTransport);

  const client = new Client({ name: "test", version: "1.0.0" });
  await client.connect(clientTransport);

  const result = await client.callTool("search_docs", { query: "authentication" });
  expect(result.content[0].text).toContain("auth");
});
```

## Publishing

### Package.json for npm
```json
{
  "name": "mcp-server-myservice",
  "version": "1.0.0",
  "bin": { "mcp-server-myservice": "./dist/index.js" },
  "scripts": { "build": "tsc", "start": "node dist/index.js" }
}
```

### Claude Desktop Config
```json
{
  "mcpServers": {
    "myservice": {
      "command": "npx",
      "args": ["-y", "mcp-server-myservice"],
      "env": { "API_KEY": "your-key" }
    }
  }
}
```
