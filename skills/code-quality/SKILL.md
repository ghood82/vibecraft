---
name: code-quality
description: "Code review, testing strategy, refactoring, tech debt. Use when asked to review code, write tests, check quality, refactor, audit, PR review, lint. Do NOT use for security scanning (use security) or performance optimization (use performance)."
---

# Code Quality

Multi-dimensional code analysis covering security, performance, correctness, and maintainability. Every finding gets a severity rating and an actionable fix.

## Review Process

1. **Scan** — Read all relevant files, understand the scope of change
2. **Analyze** — Check each dimension systematically
3. **Classify** — Rate each finding by severity
4. **Report** — Present findings with specific line references and fixes
5. **Fix** — Offer to implement fixes (don't just report, resolve)

## Review Dimensions

### Security (Critical)
- Input validation and sanitization
- Authentication and authorization
- SQL injection, XSS, CSRF
- Secrets in code
- Dependency vulnerabilities

### Performance (High)
- N+1 queries, unbounded loops
- Memory leaks, large bundle sizes
- Missing caching opportunities
- Unnecessary re-renders (React)

### Correctness (High)
- Edge cases (null, empty, overflow)
- Race conditions
- Error handling gaps
- Type safety holes

### Maintainability (Medium)
- Naming clarity
- Code duplication
- Single responsibility violations
- Missing documentation for complex logic

## Severity Classification

| Level | Icon | Meaning | Action |
|-------|------|---------|--------|
| Critical | `[CRIT]` | Security vulnerability or data loss risk | Fix immediately, block deploy |
| High | `[HIGH]` | Bug or significant performance issue | Fix before shipping |
| Medium | `[MED]` | Code smell or maintainability concern | Fix soon |
| Low | `[LOW]` | Style issue or minor improvement | Nice to have |
| Info | `[INFO]` | Observation or positive feedback | No action needed |

## Output Format

```
## Code Review Summary

**Scope**: [files reviewed]
**Overall**: [PASS / PASS WITH WARNINGS / FAIL]

### Critical Findings
[CRIT] **SQL Injection in user query** — `src/routes/users.ts:45`
- Problem: User input passed directly to SQL template literal
- Fix: Use parameterized query with `db.query()` prepared statement
- Impact: Attacker could read/modify entire database

### Positive Observations
[INFO] **Good error boundaries** — Error handling in API routes is thorough
[INFO] **Clean separation** — Business logic properly isolated from routes
```

## Testing Strategy

### What to Test
- **Always test**: Business logic, data transformations, API contracts, auth flows
- **Usually test**: Component rendering, form validation, error states
- **Skip for now**: Pure UI styling, third-party library internals, one-off scripts

### Testing Pyramid
- **Unit (60-70%)**: Pure functions, utilities, hooks, services
- **Integration (20-30%)**: API routes, database queries, component + API
- **E2E (5-10%)**: Critical user paths (signup, purchase, core workflow)

### Framework Selection
| Stack | Unit/Integration | E2E |
|-------|-----------------|-----|
| Next.js | Vitest | Playwright |
| React + Vite | Vitest | Playwright |
| Node.js API | Vitest | Supertest |
| Python | pytest | pytest + httpx |
| Cloudflare Workers | Vitest + miniflare | Playwright |

## Refactoring Patterns

When the user asks to "clean up" or "refactor":

1. **Extract** — Pull repeated logic into shared functions/hooks
2. **Simplify** — Reduce nesting, use early returns, remove dead code
3. **Type** — Add missing types, narrow any/unknown
4. **Name** — Rename unclear variables and functions
5. **Organize** — Move files to logical locations, add barrel exports
6. **Document** — Add JSDoc for non-obvious public interfaces

Read `references/review-checklist.md` for the complete checklist.
Read `references/testing-strategies.md` for detailed testing guidance.
