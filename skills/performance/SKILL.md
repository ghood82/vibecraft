---
name: performance
description: "Optimize speed, Core Web Vitals, bundle size, rendering, caching. Use for performance work. Do NOT use for security or code review."
---

# Performance Optimization

Make apps fast. Covers frontend performance (Core Web Vitals, bundle, rendering), backend (queries, caching), and measurement tooling.

## Core Web Vitals (Google's Key Metrics)

| Metric | Measures | Good | Needs Work | Poor |
|--------|----------|------|------------|------|
| **LCP** | Loading speed | < 2.5s | 2.5-4s | > 4s |
| **INP** | Responsiveness | < 200ms | 200-500ms | > 500ms |
| **CLS** | Visual stability | < 0.1 | 0.1-0.25 | > 0.25 |

## Process: Identify & Fix Performance Issues

1. **Measure**: Run Lighthouse, check DevTools Performance tab, profile bottlenecks
2. **Identify**: Is it network? Rendering? JavaScript execution? Database?
3. **Prioritize**: Fix biggest impact first (LCP before INP, frontend before backend)
4. **Implement**: Apply patterns from references for your framework
5. **Verify**: Re-measure after each change, set performance budgets

## Performance Optimization Areas

### Frontend - Loading (LCP)
- Preload critical resources (fonts, hero images)
- Optimize images (convert to WebP/AVIF, responsive sizes)
- Server-side render above-the-fold content
- Eliminate render-blocking resources
- Lazy load non-critical scripts

### Frontend - Responsiveness (INP)
- Break up long JavaScript tasks with requestIdleCallback
- Use Suspense/transitions for non-urgent state updates
- Debounce expensive operations
- Memoize expensive computations (useMemo, useCallback)
- Virtualize long lists instead of rendering all

### Frontend - Stability (CLS)
- Set explicit dimensions on images/video
- Reserve space for dynamic content (skeletons)
- Avoid injecting content above existing DOM
- Use CSS containment for isolated elements

### Bundle Optimization
- Analyze bundle size (Bundle Visualizer, source-map-explorer)
- Code split on routes and heavy components (lazy import)
- Tree shake: import specific exports, not * imports
- Set budgets: JS < 100KB gzipped, CSS < 30KB

### Caching Strategies
- Static assets (hashed) → cache forever
- API data → cache briefly (60s), revalidate in background
- User data → private cache, must revalidate
- Mutations → no-cache (POST/PUT/DELETE)

### Database & Backend
- Select only needed columns (no SELECT *)
- Add indexes for filtered/sorted columns
- Fix N+1 queries with JOINs or batch loading
- Use connection pooling for serverless
- Paginate with cursors, not OFFSET

## Reference Patterns & Code

Read `references/performance-patterns.md` for:
- LCP fix code examples (preload, image optimization, SSR)
- INP fix code (task breaking, transitions, debouncing, memoization)
- CLS fix code (dimensions, space reservation, containment)
- Bundle analysis and tree-shaking examples
- React rendering optimization patterns
- Caching headers and Next.js cache strategies
- Database query optimization examples
- Connection pooling setup
- Lighthouse CI configuration
- Performance budget enforcement

## Performance Audit Workflow

1. Run Lighthouse (Chrome DevTools)
2. Identify bottleneck (network? rendering? JS?)
3. Fix biggest issue first (highest impact, lowest effort)
4. Verify improvement (re-measure)
5. Set budgets (enforce in CI)

## Common Mistakes

- **No caching**: Cache assets, API responses, computed values
- **No code split**: Lazy load heavy components, split by route
- **Rendering all rows**: Virtualize long lists (tanstack/react-virtual)
- **Blocking resources**: Preload critical, defer non-critical
- **N+1 queries**: Use JOINs, not loops with queries
- **No performance budgets**: Regressions slip in unnoticed

---

**Next Steps**: Run Lighthouse on your app. If LCP > 2.5s, optimize images. If INP > 200ms, profile JS execution. Read references for specific patterns.
