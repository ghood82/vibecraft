---
name: research
description: "Research technologies, compare libraries, evaluate tools. Use when asked to research, compare, find the best, evaluate, what should I use, pros and cons, decision matrix. Do NOT use for codebase exploration (use codebase-understanding) or API docs lookup (use Context7 MCP)."
---

# Research & Intelligence

Investigate, compare, and recommend. Turn ambiguous questions into clear decisions with evidence.

## Research Types

### Technology Comparison
When: "Should I use X or Y?" / "What's the best Z for my use case?"

1. **Define criteria** — What matters? (performance, DX, ecosystem, bundle size, learning curve)
2. **Research each option** — Web search, docs, GitHub metrics, community sentiment
3. **Score and compare** — Build a decision matrix
4. **Recommend** — Clear recommendation with reasoning

Output: Comparison matrix + recommendation paragraph

### Best Practices Discovery
When: "How should I implement X?" / "What's the current best practice for Y?"

1. **Search** — Current documentation, blog posts, framework guides
2. **Verify** — Cross-reference multiple sources, check dates (reject outdated advice)
3. **Synthesize** — Combine into actionable guidance
4. **Contextualize** — Adapt to the user's specific stack and constraints

Output: Step-by-step implementation guide

### Architecture Research
When: "How do companies like X solve Y?" / "What's the right architecture for Z?"

1. **Pattern search** — Look for established architectural patterns
2. **Case studies** — How similar systems are built
3. **Tradeoff analysis** — Pros/cons of each approach
4. **Recommend** — Match to the user's scale and constraints

Output: Architecture recommendation with tradeoff analysis

### Library Evaluation
When: "Find me a good library for X" / "Is this library safe to use?"

Evaluate on these criteria:
- **Maintenance**: Last commit, release frequency, open issues
- **Popularity**: npm downloads, GitHub stars (as signals, not goals)
- **Quality**: TypeScript support, test coverage, documentation
- **Size**: Bundle size impact (use bundlephobia)
- **Security**: Known vulnerabilities, audit history
- **Compatibility**: Works with the user's stack

## Decision Matrix Template

```
## [Decision Title]

### Criteria (weighted)
| Criteria | Weight | Option A | Option B | Option C |
|----------|--------|----------|----------|----------|
| Performance | 30% | 4/5 | 5/5 | 3/5 |
| DX | 25% | 5/5 | 3/5 | 4/5 |
| Ecosystem | 20% | 5/5 | 4/5 | 3/5 |
| Bundle size | 15% | 3/5 | 5/5 | 4/5 |
| Learning curve | 10% | 4/5 | 2/5 | 3/5 |
| **Weighted Total** | | **4.25** | **3.95** | **3.40** |

### Recommendation
[Option A] is the best fit because [reasoning tied to the user's specific context].
Consider [Option B] if [specific scenario where it would be better].
```

## Research Quality Standards

- **Recency**: Prefer sources from the last 12 months. Flag anything older than 2 years.
- **Authority**: Official docs > framework team blog posts > community tutorials > random blogs
- **Verification**: Cross-reference claims across multiple sources
- **Specificity**: Don't say "it's fast" — say "handles 10k requests/sec on a single core"
- **Honesty**: If the research is inconclusive, say so. Don't force a recommendation.

## Memory Integration

After completing research, store decisions in memory:
- "Chose Drizzle over Prisma for [reason]"
- "User prefers lightweight libraries over feature-rich ones"
- "Project constraint: must work on Cloudflare Workers (no Node.js APIs)"

Read `references/research-methods.md` for evaluation frameworks and templates.
