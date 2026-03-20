# Getting Started with VibeCraft

This guide walks you through installing VibeCraft, running your first project, and understanding how the plugin works under the hood.

## Installation

1. Download `vibecraft.plugin` (or build it from source).
2. In your terminal, install it into Claude Code:

```bash
claude plugin add vibecraft.plugin
```

3. Start a new Claude Code session. You should see the VibeCraft banner:

```
## VibeCraft v0.7.0 Active
Skills (33): orchestrator, vibe-coding, code-quality, ...
Agents (5): code-reviewer, security-scanner, researcher, qa-tester, ux-auditor
Commands (10): /vibe, /ship, /review, /research, /plan, /secure, /test, /improve, /status, /feature
```

That's it — no API keys, no config files, no environment variables. VibeCraft works with whatever MCPs and tools you already have connected.

## Your First Project

Let's scaffold a full-stack app from scratch:

```
/vibe a Next.js app with Supabase auth, Stripe billing, and a dashboard
```
VibeCraft's orchestrator will:

1. **Detect intent** — recognizes this as a scaffolding + multi-service integration request
2. **Route to skills** — engages `project-scaffolding` for the base structure, `vibe-coding` for code generation, `database` for Supabase schema, and `api-design` for Stripe endpoints
3. **Check your MCPs** — if you have the Vercel MCP connected, it'll set up the project for instant deployment
4. **Generate everything** — project structure, typed config, auth flows, billing integration, dashboard components
5. **Update memory** — remembers your stack preferences for next time

## Key Commands to Try

| Command | Try This |
|---------|----------|
| `/vibe` | `/vibe a CLI tool that converts CSV to JSON` |
| `/review` | `/review src/` — get a full code review |
| `/ship` | `/ship vercel` — deploy with pre-flight checks |
| `/secure` | `/secure` — run an OWASP security scan |
| `/test` | `/test src/utils/` — generate and run tests |
| `/research` | `/research best React state management in 2025` |
| `/plan` | `/plan migrate from REST to tRPC` |
| `/feature` | `/feature add dark mode with system preference detection` |
| `/status` | `/status` — see project health dashboard |
| `/improve` | `/improve` — self-evaluate and learn from this session |

## How the Orchestrator Routes Requests

You don't need to think about which skill to use. Just describe what you want in natural language, and the orchestrator figures out the rest.

The orchestrator uses a routing guide that maps intent patterns to skills. For example:
- "build", "create", "scaffold" → `vibe-coding` or `project-scaffolding`
- "review", "check quality" → `code-quality`
- "deploy", "ship", "push to prod" → `deployment`
- "is this secure", "vulnerabilities" → `security`
- "compare X vs Y", "which should I use" → `research`
When multiple skills are needed, the orchestrator coordinates them in parallel where possible. For a request like "build an API with auth, add tests, and deploy it," it will run `api-design` → `code-quality` → `deployment` in sequence, checking each stage before proceeding.

## How Memory and Learning Work

### Memory

VibeCraft remembers your preferences across sessions using a two-tier system:

- **CLAUDE.md** is the hot cache — it stores your top preferences, active projects, tech stack, and common terms. Claude reads this at the start of every session.
- **memory/** is deep storage — full project history, detailed preferences, architectural decisions, and past conversation context.

You don't need to configure memory. It builds up naturally as you use VibeCraft. After a few sessions, Claude will know your preferred frameworks, coding style, deployment targets, and naming conventions.

### Learning

The **learning** skill tracks patterns across sessions:
- Which approaches worked well and which caused issues
- Recurring error patterns and their fixes
- Your feedback on output quality

Over time, VibeCraft adapts its behavior to avoid past mistakes and double down on what works for you.

## Tips for Getting the Most Out of VibeCraft

1. **Be specific about what you want.** "Build a REST API" is good. "Build a REST API with Express, Drizzle ORM, Postgres, JWT auth, and rate limiting" is better. VibeCraft will scaffold more accurately with more context.

2. **Use `/improve` regularly.** Running self-evaluation after each session feeds the learning loop and makes future sessions better.

3. **Let the orchestrator route.** You don't need to name specific skills. Just describe your goal and let VibeCraft figure out the routing.

4. **Connect your MCPs.** VibeCraft gets significantly more powerful with MCPs connected. Vercel, Cloudflare, Figma, Jira, and Context7 all unlock additional capabilities.

5. **Check `/status` periodically.** The project health dashboard gives you a quick overview of code quality, test coverage, security posture, and deployment status.

6. **Use `/plan` for big tasks.** Before diving into a large feature, run `/plan` to get a prioritized breakdown. VibeCraft will create a structured execution plan.

7. **Trust the memory.** If you've told VibeCraft something once (preferred stack, naming conventions, deployment target), it remembers. You don't need to repeat yourself.