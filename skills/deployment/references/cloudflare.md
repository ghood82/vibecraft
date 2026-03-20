# Cloudflare Deployment Reference

## Workers Configuration

### wrangler.toml
```toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"
compatibility_flags = ["nodejs_compat"]

# Environment-specific
[env.staging]
name = "my-worker-staging"
vars = { ENVIRONMENT = "staging" }

[env.production]
name = "my-worker"
vars = { ENVIRONMENT = "production" }
routes = [{ pattern = "api.example.com/*", zone_name = "example.com" }]

# D1 Database
[[d1_databases]]
binding = "DB"
database_name = "my-db"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# R2 Object Storage
[[r2_buckets]]
binding = "BUCKET"
bucket_name = "my-bucket"

# KV Namespace
[[kv_namespaces]]
binding = "CACHE"
id = "xxxxxxxx"
```

## D1 Database

### Setup
```bash
# Create database
wrangler d1 create my-db

# Run migrations
wrangler d1 migrations apply my-db

# Execute SQL directly
wrangler d1 execute my-db --command "SELECT * FROM users LIMIT 5"
```

### Migrations
```bash
# Create migration
wrangler d1 migrations create my-db "create_users_table"
```

```sql
-- migrations/0001_create_users_table.sql
CREATE TABLE IF NOT EXISTS users (
  id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
```

### Querying in Code
```typescript
interface Env {
  DB: D1Database;
  BUCKET: R2Bucket;
  CACHE: KVNamespace;
}

// Prepared statements (ALWAYS use these, never template literals)
const user = await env.DB.prepare("SELECT * FROM users WHERE id = ?")
  .bind(userId)
  .first<User>();

// Batch queries
const results = await env.DB.batch([
  env.DB.prepare("INSERT INTO users (name, email) VALUES (?, ?)").bind(name, email),
  env.DB.prepare("INSERT INTO audit_log (action) VALUES (?)").bind("user_created"),
]);
```

## R2 Object Storage

```typescript
// Upload
await env.BUCKET.put(key, body, {
  httpMetadata: { contentType: "image/png" },
  customMetadata: { uploadedBy: userId },
});

// Download
const object = await env.BUCKET.get(key);
if (!object) return new Response("Not found", { status: 404 });

return new Response(object.body, {
  headers: { "Content-Type": object.httpMetadata?.contentType ?? "application/octet-stream" },
});

// List
const list = await env.BUCKET.list({ prefix: "uploads/", limit: 100 });

// Delete
await env.BUCKET.delete(key);
```

## KV Namespace

```typescript
// Set with TTL (seconds)
await env.CACHE.put("session:abc", JSON.stringify(sessionData), { expirationTtl: 3600 });

// Get
const data = await env.CACHE.get("session:abc", "json");

// Delete
await env.CACHE.delete("session:abc");

// List keys
const keys = await env.CACHE.list({ prefix: "session:" });
```

## Pages Deployment

```bash
# Deploy a static site or framework
wrangler pages deploy ./dist

# With functions (server-side rendering)
# Place functions in /functions directory
```

## Secrets Management

```bash
# Set a secret (not visible in wrangler.toml)
wrangler secret put API_KEY
wrangler secret put API_KEY --env production

# List secrets
wrangler secret list
```

## Wrangler CLI Commands

```bash
# Local development
wrangler dev

# Deploy
wrangler deploy
wrangler deploy --env staging
wrangler deploy --env production

# Tail logs in real-time
wrangler tail

# Rollback
wrangler rollback

# Database
wrangler d1 list
wrangler d1 execute DB_NAME --command "SQL here"
wrangler d1 migrations apply DB_NAME
```

## Performance Tips

- Keep worker bundle small (< 1MB ideally)
- Use KV for caching hot data (reads are very fast)
- Use D1 for relational data (keep queries simple)
- Use R2 for large objects (images, files)
- Minimize external API calls (add latency)
- Use `waitUntil()` for non-critical async work after response
- Enable Smart Placement for CPU-intensive workers

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Worker exceeds CPU limit | Complex computation | Optimize code, use Smart Placement |
| D1 query slow | Missing index | Add appropriate indexes |
| KV data stale | Eventual consistency | Use shorter TTL or check freshness |
| Worker bundle too large | Too many dependencies | Tree-shake, use smaller libraries |
| CORS errors | Missing headers | Add CORS middleware in Hono |
