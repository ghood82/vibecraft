# OWASP Top 10 Reference

Practical detection patterns and fixes for each vulnerability class.

## A01: Broken Access Control

**What**: Users accessing data or functionality they shouldn't.

**Detection Patterns**:
- API endpoints without auth middleware
- Missing ownership checks (`WHERE user_id = currentUser.id`)
- Direct object references without authorization (`/api/users/123/settings`)
- Admin functions accessible to regular users
- CORS misconfiguration (`Access-Control-Allow-Origin: *`)

**Fix**:
```typescript
// Middleware: verify auth on every protected route
app.use("/api/*", async (c, next) => {
  const session = await getSession(c);
  if (!session) return c.json({ error: "Unauthorized" }, 401);
  c.set("userId", session.userId);
  await next();
});

// Route: verify ownership
app.get("/api/documents/:id", async (c) => {
  const doc = await db.query.documents.findFirst({
    where: and(eq(documents.id, c.req.param("id")), eq(documents.userId, c.get("userId"))),
  });
  if (!doc) return c.json({ error: "Not found" }, 404);
  return c.json(doc);
});
```

## A02: Cryptographic Failures

**What**: Weak encryption, plaintext secrets, outdated algorithms.

**Detection Patterns**:
- Passwords stored in plaintext or with MD5/SHA1
- Sensitive data transmitted over HTTP
- Hardcoded encryption keys
- Weak random number generation (`Math.random()` for security)

**Fix**:
```typescript
import { hash, verify } from "@node-rs/argon2";

// Hash password
const hashed = await hash(password, { memoryCost: 65536, timeCost: 3 });

// Verify password
const valid = await verify(hashed, password);

// Secure random values
const token = crypto.randomUUID();
const bytes = crypto.getRandomValues(new Uint8Array(32));
```

## A03: Injection

**What**: Untrusted data sent to an interpreter (SQL, NoSQL, OS, LDAP).

**Detection Patterns**:
- String concatenation in SQL queries
- Template literals with user input in queries
- `eval()` or `new Function()` with user input
- Shell command execution with user input

**Fix**: Always use parameterized queries, prepared statements, or ORMs.

## A04: Insecure Design

**What**: Architecture-level flaws, missing security controls by design.

**Detection Patterns**:
- No rate limiting on sensitive endpoints
- No account lockout after failed attempts
- Business logic that trusts client-side validation only
- No audit logging for sensitive operations

**Fix**: Design security in from the start:
- Rate limit all auth endpoints
- Validate on server, never trust client
- Log all sensitive operations
- Implement principle of least privilege

## A05: Security Misconfiguration

**What**: Default configs, unnecessary features, verbose errors.

**Detection Patterns**:
- Default credentials or configurations
- Unnecessary HTTP methods enabled
- Directory listing enabled
- Stack traces in production error responses
- Missing security headers

**Fix**:
```typescript
// Production error handler — never expose internals
app.onError((err, c) => {
  console.error(err); // Log internally
  return c.json({ error: "Internal server error" }, 500); // Generic to client
});
```

## A06: Vulnerable Components

**What**: Using components with known vulnerabilities.

**Fix**:
```bash
npm audit fix
npm update
npx npm-check-updates -u  # Check for major updates
```

## A07: Authentication Failures

**What**: Weak auth implementation allowing account compromise.

**Detection Patterns**:
- No password complexity requirements
- Missing brute-force protection
- Session tokens in URLs
- No MFA option for sensitive operations
- Credentials transmitted in plain text

## A08: Software and Data Integrity Failures

**What**: Code and infrastructure that doesn't verify integrity.

**Detection Patterns**:
- CDN scripts without integrity hashes
- Unsigned software updates
- Deserialization of untrusted data
- CI/CD pipelines without verification

**Fix**:
```html
<!-- Use Subresource Integrity for CDN scripts -->
<script src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"></script>
```

## A09: Security Logging and Monitoring Failures

**What**: Insufficient logging to detect and respond to breaches.

**Fix**: Log all auth events, access control failures, input validation failures, and admin actions. Include timestamp, user ID, IP, action, and outcome.

## A10: Server-Side Request Forgery (SSRF)

**What**: Application fetches a URL supplied by the user without validation.

**Detection Patterns**:
- User-supplied URLs passed to `fetch()` or HTTP clients
- URL parameters that control server-side requests
- Webhook URLs without validation

**Fix**:
```typescript
// Validate URLs before fetching
function isAllowedUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    // Block internal networks
    if (parsed.hostname === "localhost" || parsed.hostname.startsWith("192.168.") || parsed.hostname.startsWith("10.") || parsed.hostname === "127.0.0.1") {
      return false;
    }
    // Only allow HTTPS
    return parsed.protocol === "https:";
  } catch {
    return false;
  }
}
```
