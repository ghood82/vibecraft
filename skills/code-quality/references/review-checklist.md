# Code Review Checklist

Use this systematically when reviewing code. Check each category, document findings with severity.

## Security

### Input Validation
- [ ] All user inputs validated with Zod or equivalent
- [ ] File uploads restricted by type and size
- [ ] URL parameters and query strings sanitized
- [ ] Request body size limits enforced

### Injection Prevention
- [ ] SQL queries use parameterized statements (never template literals with user input)
- [ ] No `eval()`, `new Function()`, or dynamic code execution with user input
- [ ] HTML output escaped or uses framework auto-escaping (React JSX, etc.)
- [ ] Command injection prevented (no `exec()` with user input)

### Authentication & Authorization
- [ ] Auth required on all non-public endpoints
- [ ] Role-based access control enforced server-side
- [ ] JWT tokens have reasonable expiration
- [ ] Password hashing uses bcrypt/argon2 (not MD5/SHA)
- [ ] Session management is secure (httpOnly, secure, sameSite cookies)

### Secrets
- [ ] No hardcoded API keys, tokens, or passwords
- [ ] Environment variables used for all secrets
- [ ] `.env` files in `.gitignore`
- [ ] No secrets logged or returned in responses

### Dependencies
- [ ] `npm audit` or equivalent shows no critical vulnerabilities
- [ ] No unnecessary dependencies (check bundle impact)
- [ ] Lock file committed (package-lock.json or pnpm-lock.yaml)

## Performance

### Database
- [ ] No N+1 query patterns (use joins or batch loading)
- [ ] Queries filtered at the database level, not in application code
- [ ] Appropriate indexes exist for query patterns
- [ ] Pagination implemented for list endpoints
- [ ] No `SELECT *` — only fetch needed columns

### Memory & CPU
- [ ] No unbounded arrays or object growth
- [ ] Large datasets processed in streams/chunks
- [ ] No synchronous file operations in hot paths
- [ ] Event listeners and subscriptions cleaned up on unmount
- [ ] No memory leaks from closures holding large references

### Frontend
- [ ] Images optimized (next/image, WebP, lazy loading)
- [ ] Bundle size reasonable (check with `next build` or `vite build`)
- [ ] No unnecessary re-renders (React.memo, useMemo, useCallback where appropriate)
- [ ] Heavy computation offloaded (Web Workers, server-side)
- [ ] Code splitting for large components/routes

### Caching
- [ ] Static assets have cache headers
- [ ] API responses cached where appropriate (TanStack Query, SWR)
- [ ] Database query results cached when read-heavy

## Correctness

### Error Handling
- [ ] All async operations have try/catch or .catch()
- [ ] Error responses include useful messages (not stack traces in production)
- [ ] Failed operations don't leave partial state
- [ ] Network errors handled gracefully in the UI

### Edge Cases
- [ ] Empty arrays/objects handled
- [ ] Null/undefined checks before property access
- [ ] Integer overflow and boundary values considered
- [ ] Unicode and special characters handled in user input
- [ ] Concurrent access patterns considered (race conditions)

### Type Safety
- [ ] No `any` types (use `unknown` and narrow)
- [ ] Zod schemas match database schema / API contract
- [ ] Discriminated unions used for state variants
- [ ] Return types explicitly defined on exported functions

## Maintainability

### Naming
- [ ] Variables and functions describe what they do, not how
- [ ] Boolean variables start with is/has/should/can
- [ ] Functions start with verbs (get, create, update, delete, validate)
- [ ] No abbreviations unless universally understood (id, url, api)

### Structure
- [ ] Each file has a single clear responsibility
- [ ] Files under 300 lines (split if larger)
- [ ] Related code colocated (component + test + types)
- [ ] Circular dependencies avoided

### Documentation
- [ ] Complex business logic has inline comments explaining WHY
- [ ] Public APIs have JSDoc/docstring
- [ ] Non-obvious regex patterns explained
- [ ] Magic numbers replaced with named constants

## Accessibility

- [ ] Interactive elements are keyboard accessible
- [ ] Images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator of state
- [ ] Focus management handled for modals/dialogs
- [ ] Semantic HTML used (nav, main, article, button vs div)
