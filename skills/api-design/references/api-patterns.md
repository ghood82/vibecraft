# Advanced API Patterns

## Authentication Patterns

### API Key
```
Authorization: Bearer sk_live_abc123
```
Best for: Server-to-server, internal APIs, simple integrations.

### JWT (JSON Web Token)
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```
Best for: Stateless auth, microservices, mobile apps.

### OAuth 2.0
Best for: Third-party access, user-scoped permissions, social login.
Flows: Authorization Code (web), PKCE (SPA/mobile), Client Credentials (server-to-server).

### Session Cookies
Best for: Traditional web apps, server-rendered apps.

## Idempotency

For safe retries on network failures:
```
POST /api/v1/payments
Idempotency-Key: unique-request-id-123

// Server: check if this key was already processed
// If yes: return the original response
// If no: process and store the result keyed by idempotency key
```

## Webhook Design

```
POST https://your-app.com/webhooks/stripe

Headers:
  Stripe-Signature: t=123,v1=abc...

Body:
{
  "id": "evt_123",
  "type": "payment_intent.succeeded",
  "data": { "object": { ... } },
  "created": 1711000000
}
```

**Webhook best practices:**
- Verify signatures to prevent spoofing
- Return 200 quickly, process async
- Implement retry handling (idempotent processing)
- Log all webhook events for debugging
- Use a queue for processing (don't block the response)

## CORS Configuration

```typescript
// Hono
import { cors } from "hono/cors";

app.use("/api/*", cors({
  origin: ["https://myapp.com", "https://staging.myapp.com"],
  allowMethods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
  allowHeaders: ["Content-Type", "Authorization"],
  maxAge: 86400,
}));
```

## API Error Handling Strategy

### Error Code Registry
Define all possible errors in one place:

```typescript
const API_ERRORS = {
  VALIDATION_ERROR: { status: 400, message: "Invalid input" },
  UNAUTHORIZED: { status: 401, message: "Authentication required" },
  FORBIDDEN: { status: 403, message: "Insufficient permissions" },
  NOT_FOUND: { status: 404, message: "Resource not found" },
  CONFLICT: { status: 409, message: "Resource already exists" },
  RATE_LIMITED: { status: 429, message: "Too many requests" },
  INTERNAL_ERROR: { status: 500, message: "Internal server error" },
} as const;
```

### Structured Error Response
```typescript
class APIError extends Error {
  constructor(
    public code: keyof typeof API_ERRORS,
    public details?: Record<string, string>[],
  ) {
    super(API_ERRORS[code].message);
  }

  toResponse() {
    const { status, message } = API_ERRORS[this.code];
    return Response.json(
      { error: { code: this.code, message, details: this.details } },
      { status }
    );
  }
}

// Usage
throw new APIError("VALIDATION_ERROR", [
  { field: "email", message: "Invalid email format" },
]);
```

## Bulk Operations

```
POST /api/v1/users/bulk
{
  "operations": [
    { "method": "create", "data": { "name": "Alice" } },
    { "method": "update", "id": "123", "data": { "name": "Bob" } },
    { "method": "delete", "id": "456" }
  ]
}

Response:
{
  "results": [
    { "index": 0, "status": "success", "data": { "id": "789" } },
    { "index": 1, "status": "success", "data": { "id": "123" } },
    { "index": 2, "status": "error", "error": { "code": "NOT_FOUND" } }
  ]
}
```

## Caching Headers

```typescript
// Static data (cache aggressively)
res.headers.set("Cache-Control", "public, max-age=3600, s-maxage=86400");

// User-specific data (cache privately)
res.headers.set("Cache-Control", "private, max-age=60");

// Never cache (auth endpoints, mutations)
res.headers.set("Cache-Control", "no-store");

// ETag for conditional requests
res.headers.set("ETag", `"${hash(data)}"`);
```

## API Monitoring Checklist

- Response time percentiles (p50, p95, p99)
- Error rate by endpoint
- Rate limit hits
- Request volume trends
- Authentication failures
- Slow queries (correlate with DB monitoring)
