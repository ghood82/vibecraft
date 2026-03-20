# Research Methods

## Technology Evaluation Framework

### The 5-Dimension Score

Rate each option 1-5 on these dimensions:

1. **Capability** — Does it do what we need? Features, API completeness, extensibility
2. **Developer Experience** — API ergonomics, documentation quality, TypeScript support, error messages
3. **Ecosystem** — Community size, plugins/extensions, hiring pool, StackOverflow answers
4. **Performance** — Speed, bundle size, memory usage, scalability characteristics
5. **Stability** — Maturity, breaking change frequency, backing (company vs individual), funding

### Weight by Context

| Project Type | Capability | DX | Ecosystem | Performance | Stability |
|-------------|-----------|-----|-----------|-------------|-----------|
| Prototype/MVP | 30% | 30% | 15% | 10% | 15% |
| Production App | 25% | 20% | 20% | 15% | 20% |
| High-Traffic | 20% | 15% | 15% | 30% | 20% |
| Enterprise | 20% | 15% | 20% | 15% | 30% |

## Library Quality Signals

### Green Flags
- Regular commits in last 3 months
- Semantic versioning with changelogs
- Comprehensive TypeScript types (ideally built-in, not @types)
- Test suite with good coverage
- Clear migration guides between major versions
- Active issue triage (issues get responses)
- Used by well-known projects

### Red Flags
- No commits in 6+ months with open issues
- No TypeScript support
- No tests
- Single maintainer with no succession plan
- Breaking changes without semver major bumps
- Abandoned PRs and unanswered issues

### Metrics to Check
```
npm: weekly downloads, download trend (growing/declining)
GitHub: stars, forks, open issues, last commit, contributors
Bundlephobia: minified size, gzipped size, tree-shakeable?
Snyk: known vulnerabilities
```

## Comparison Report Template

```markdown
# [Topic] Comparison

## Context
[What we're trying to decide and why]

## Options Evaluated
1. **[Option A]** — [one-line description]
2. **[Option B]** — [one-line description]
3. **[Option C]** — [one-line description]

## Comparison Matrix
[Decision matrix with weighted scores]

## Deep Dive

### [Option A]
**Strengths**: [list]
**Weaknesses**: [list]
**Best for**: [scenarios]
**Watch out for**: [gotchas]

### [Option B]
[Same structure]

## Recommendation
[Clear recommendation with reasoning]

## Sources
[Links to docs, benchmarks, and articles consulted]
```

## Source Quality Assessment

| Tier | Source Type | Trust Level |
|------|-----------|-------------|
| 1 | Official documentation | High — canonical source |
| 2 | Framework team blog posts | High — authoritative |
| 3 | Well-known tech blogs (Vercel, Cloudflare, etc.) | Medium-High |
| 4 | Conference talks and tutorials | Medium — check date |
| 5 | Community blog posts | Medium-Low — verify claims |
| 6 | StackOverflow answers | Low-Medium — check votes and date |
| 7 | AI-generated content | Low — always verify |

## Common Research Mistakes

- **Recency bias**: Newest isn't always best. Proven > trendy.
- **Popularity bias**: npm downloads don't equal quality.
- **Benchmark worship**: Synthetic benchmarks rarely reflect real-world performance.
- **Feature comparison without context**: More features ≠ better for your use case.
- **Ignoring migration cost**: The best technology is the one your team already knows, unless there's a compelling reason to switch.
