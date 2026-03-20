# Security Checklist

Comprehensive pre-deployment and ongoing security checklist.

## Pre-Deployment Security Checklist

### Authentication
- [ ] Passwords hashed with bcrypt (cost 12+) or argon2
- [ ] Rate limiting on login endpoints (10 attempts/min/IP)
- [ ] Account lockout after repeated failures
- [ ] Secure session management (httpOnly, secure, sameSite cookies)
- [ ] JWT tokens expire within reasonable time (15min access, 7d refresh)
- [ ] Refresh token rotation implemented
- [ ] Password reset flow uses time-limited, single-use tokens
- [ ] MFA available for admin/sensitive accounts

### Authorization
- [ ] Server-side authorization on every endpoint
- [ ] Ownership checks on all resource access
- [ ] Role-based access control (not just role checks in UI)
- [ ] Admin endpoints separated and double-verified
- [ ] API keys scoped to minimum required permissions

### Input Validation
- [ ] All inputs validated server-side with Zod or equivalent
- [ ] File uploads restricted by type, size, and content
- [ ] SQL queries parameterized (no string concatenation)
- [ ] HTML output escaped (framework default or sanitizer)
- [ ] URL parameters validated and typed

### Secrets
- [ ] No hardcoded secrets in source code
- [ ] All secrets in environment variables
- [ ] `.env` files in `.gitignore`
- [ ] Different secrets per environment
- [ ] Secret rotation plan documented

### Headers & Transport
- [ ] HTTPS enforced (HSTS header set)
- [ ] Security headers configured (CSP, X-Frame-Options, etc.)
- [ ] CORS restricted to known origins
- [ ] Cookies marked secure, httpOnly, sameSite
- [ ] API responses don't leak server info (X-Powered-By removed)

### Dependencies
- [ ] `npm audit` shows no critical/high vulnerabilities
- [ ] Lock file committed
- [ ] Dependencies pinned to specific versions
- [ ] Regular update schedule established

### Data Protection
- [ ] Sensitive data encrypted at rest
- [ ] PII minimized (don't collect what you don't need)
- [ ] Database backups encrypted
- [ ] Logs don't contain sensitive data (passwords, tokens, PII)
- [ ] Error responses don't leak internal details

### API Security
- [ ] Rate limiting on all public endpoints
- [ ] Request size limits configured
- [ ] Pagination enforced (no unbounded queries)
- [ ] API versioning in place
- [ ] Webhook signatures validated

### Infrastructure
- [ ] Firewall rules restrict access to necessary ports only
- [ ] Database not publicly accessible
- [ ] SSH key auth only (no password auth)
- [ ] Monitoring and alerting configured
- [ ] Incident response plan documented

## Periodic Security Tasks

### Weekly
- [ ] Review dependency vulnerability alerts
- [ ] Check error logs for unusual patterns
- [ ] Review access logs for anomalies

### Monthly
- [ ] Run full dependency audit
- [ ] Review and rotate any expiring secrets
- [ ] Check for new OWASP advisories

### Quarterly
- [ ] Full security review of new code
- [ ] Penetration testing (manual or automated)
- [ ] Review and update security documentation
- [ ] Verify backup restoration process
