---
name: api-design
description: "API design patterns (REST, GraphQL, tRPC, WebSocket, rate limiting). Use for API work. Do NOT use for database schema, deployment, or frontend UI components."
---

# API Design

Design clean, consistent, well-documented APIs. Covers REST, GraphQL, tRPC, and WebSocket patterns.

## When to Use What

| Pattern | Best For |
|---------|----------|
| **REST** | Public APIs, multi-language clients, standard CRUD |
| **GraphQL** | Complex nested data, multiple clients needing different shapes |
| **tRPC** | Full-stack TypeScript, end-to-end type safety |
| **WebSocket** | Real-time features (chat, notifications, live updates) |

## REST API Quick Reference

### URL Structure
```
GET    /api/v1/users           # List
POST   /api/v1/users           # Create
GET    /api/v1/users/:id       # Read
PATCH  /api/v1/users/:id       # Update (partial)
DELETE /api/v1/users/:id       # Delete
POST   /api/v1/users/:id/verify  # Actions (when CRUD doesn't fit)
```

**Rules**: Plural nouns, kebab-case, no verbs in URLs, max 2 levels of nesting.

### Status Codes
200 Success, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Rate Limited, 500 Server Error.

### Pagination (Cursor-Based Recommended)
```
GET /api/v1/users?cursor=abc123&limit=20
→ { "data": [...], "nextCursor": "def456", "hasMore": true }
```

### Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [{ "field": "email", "message": "Must be a valid email" }]
  }
}
```

## tRPC (TypeScript-First)

End-to-end type safety without code generation. Define procedures with Zod schemas, call from client with full autocomplete.

## Rate Limiting

Use `@upstash/ratelimit` with Redis for sliding window rate limiting. Return 429 with `Retry-After` header.

## API Versioning

URL path recommended: `/api/v1/users`. Alternatives: Accept header, query param.

## OpenAPI / Swagger

Generate from code using `zod-openapi` or write manually with Swagger UI.

## Reference Docs

Read `references/api-patterns.md` for advanced REST patterns and middleware.
Read `references/stripe-patterns.md` for Stripe checkout, subscriptions, webhooks, and billing portal integration.
