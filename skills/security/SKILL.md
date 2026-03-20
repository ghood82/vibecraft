---
name: security
description: "Security scan, OWASP vulnerabilities, hardening, audit. Use when asked to secure, scan for vulnerabilities, harden, OWASP check, security audit, check for injection, check auth. Do NOT use for code review (use code-quality) or secrets management (use env-secrets)."
---

# Security

Scan, detect, and fix security vulnerabilities. Every finding gets a severity rating, impact assessment, and concrete fix.

## Scan Workflow

1. **Dependency scan** — Check for known vulnerabilities in packages
2. **Secret detection** — Scan for hardcoded keys, tokens, passwords
3. **Code analysis** — OWASP Top 10 patterns in source code
4. **Configuration review** — Security headers, CORS, CSP, rate limiting
5. **Authentication review** — Auth flow, session management, token handling
6. **Report** — Findings with severity, impact, and remediation

## Severity Classification

| Level | Criteria | Response |
|-------|----------|----------|
| **Critical** | Remote code execution, data breach, auth bypass | Fix immediately, block deploy |
| **High** | XSS, SQL injection, privilege escalation | Fix before next deploy |
| **Medium** | CSRF, insecure config, info disclosure | Fix within sprint |
| **Low** | Missing headers, verbose errors | Fix when convenient |
| **Info** | Best practice suggestion | Consider for improvement |

## Quick Scans

### Dependency Vulnerability Check
```bash
# Node.js
npm audit
npm audit --audit-level=high

# Python
pip-audit
safety check

# General
npx snyk test
```

### Secret Detection
Look for patterns in code:
- API keys: `sk-`, `pk_`, `AKIA`, `ghp_`, `xox[bpas]-`
- Generic secrets: `password=`, `secret=`, `token=`, `key=` followed by string literals
- Private keys: `-----BEGIN RSA PRIVATE KEY-----`
- Connection strings: `postgres://`, `mongodb://`, `redis://` with credentials

### Security Headers Check
Verify these headers are set:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'; script-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## Common Vulnerability Patterns

### Injection Prevention
```typescript
// BAD — SQL injection
const user = await db.execute(`SELECT * FROM users WHERE id = '${userId}'`);

// GOOD — Parameterized query
const user = await db.prepare("SELECT * FROM users WHERE id = ?").bind(userId).first();
```

### XSS Prevention
```typescript
// BAD — Inserting raw HTML
element.innerHTML = userInput;

// GOOD — React auto-escapes by default
return <div>{userInput}</div>;

// GOOD — Explicit sanitization when HTML is needed
import DOMPurify from "dompurify";
const clean = DOMPurify.sanitize(userInput);
```

### Auth Best Practices
- Hash passwords with bcrypt (cost factor 12+) or argon2
- Use httpOnly, secure, sameSite cookies for sessions
- Implement rate limiting on auth endpoints
- Add CSRF tokens for state-changing operations
- Use short-lived JWTs with refresh token rotation

## Output Format

```
## Security Scan Report

**Scan Date**: [date]
**Scope**: [files/directories scanned]
**Overall Risk**: [CRITICAL / HIGH / MEDIUM / LOW / CLEAN]

### Findings

[CRIT] **Hardcoded API key in source** — `src/lib/api.ts:12`
- Impact: Attacker gains access to third-party service
- Fix: Move to environment variable, rotate the exposed key immediately
- Effort: 5 minutes

[HIGH] **Missing rate limiting on /api/auth/login** — `src/routes/auth.ts`
- Impact: Brute force attacks possible
- Fix: Add rate limiter middleware (10 attempts per minute per IP)
- Effort: 15 minutes

### Summary
| Severity | Count |
|----------|-------|
| Critical | 1 |
| High | 1 |
| Medium | 0 |
| Low | 2 |
```

Read `references/owasp-top-10.md` for detailed vulnerability patterns and fixes.
Read `references/security-checklist.md` for the complete pre-deployment security checklist.
