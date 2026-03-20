# VibeCraft

**The all-in-one agentic system for vibe coding on Claude Code.**

VibeCraft transforms Claude into a full-stack development partner with 34 skills, 5 specialized agents, 10 slash commands, and a multi-layered security system. It combines project scaffolding, workflow orchestration, multi-agent pipelines, code quality, deployment, security, research, UI/UX design, AI/agent building, performance optimization, monitoring, database management, compliance, persistent memory, and self-improvement into a single cohesive plugin.

## What It Does

- **Scaffold projects** from scratch across any stack (Next.js, Cloudflare Workers, React, Python, and more)
- **Generate production code** with proper types, error handling, and patterns
- **Review code** across security, performance, correctness, and maintainability
- **Deploy anywhere** — Vercel, Cloudflare, Docker, Railway, Fly.io
- **Research technologies** with structured comparison matrices and recommendations
- **Design UIs** following professional design principles and WCAG accessibility
- **Scan for security** vulnerabilities using OWASP Top 10 framework
- **Enforce compliance** — HIPAA, SOC 2, GDPR, PCI DSS at the code level
- **Write tests** — unit, integration, and E2E with Playwright
- **Design APIs** — REST, GraphQL, tRPC, WebSocket patterns- **Build AI agents** — MCP servers, tool use, RAG pipelines, prompt engineering
- **Optimize performance** — Core Web Vitals, bundle analysis, caching, rendering
- **Monitor production** — error tracking, structured logging, health checks, alerting
- **Manage databases** — Drizzle/Prisma migrations, schema design, Supabase, seeding
- **Handle secrets** — Vault, AWS/GCP/Azure secret managers, SOPS, rotation policies
- **Configure linting** — Biome, ESLint 9, Prettier, TypeScript strict, pre-commit hooks
- **Generate CI/CD pipelines** — GitHub Actions, preview deploys, release automation
- **Manage monorepos** — Turborepo, Nx, pnpm workspaces, package boundaries
- **Orchestrate multi-agent workflows** — coordinated pipelines across 5 specialized agents
- **Defend against prompt injection** — scans fetched content and commands for attacks
- **Run iterative fix loops** — test→fix→retest cycles until CI goes green
- **Schedule automation** — cron jobs, recurring tasks, dependency update schedules
- **Manage git workflows** — conventional commits, branching strategies, PR best practices
- **Remember your preferences** across sessions with persistent and semantic memory
- **Self-evaluate and improve** its own output quality over time
- **Generate documentation** — READMEs, API docs, architecture docs, changelogs

## Quick Start

Install the plugin and start using it immediately — no environment variables or external services required. VibeCraft uses whatever tools and MCPs you already have connected.

```
/vibe a Next.js SaaS with Stripe billing     # scaffold a full project
/ship vercel                                   # deploy with pre-flight checks
/review src/                                   # full code review
/secure                                        # OWASP security scan
/status                                        # project health dashboard
```

See [GETTING-STARTED.md](GETTING-STARTED.md) for a full walkthrough.
## Commands

| Command | Description |
|---------|-------------|
| `/vibe [what to build]` | Start a vibe coding session — scaffold, build, prototype |
| `/ship [platform]` | Deploy to production with pre-flight checks |
| `/review [file/dir]` | Full code review across 4 dimensions |
| `/research [topic]` | Research and compare technologies |
| `/plan [goal]` | GSD mode — plan, prioritize, and execute |
| `/secure [file/dir]` | Security scan with OWASP framework |
| `/test [file/feature]` | Generate and run tests |
| `/improve` | Self-evaluate this session and update memory |
| `/status` | Project health dashboard |
| `/feature [what to build]` | Structured 7-phase feature development workflow |

## Multi-Agent Architecture

VibeCraft includes 5 specialized agents that can be spawned for deep, focused analysis. Unlike skills (which provide quick, lightweight guidance), agents run as dedicated subprocesses for thorough multi-file work.

### Agents

| Agent | Color | Specialty | When to Use |
|-------|-------|-----------|-------------|
| **code-reviewer** | Blue | Multi-dimensional code analysis | Pre-merge reviews, full codebase audits, multi-file quality analysis |
| **security-scanner** | Red | OWASP vulnerability detection | Pre-deploy security gates, vulnerability audits, auth code review |
| **researcher** | Cyan | Technology research and comparison | Multi-source research, technology evaluations, architectural decisions |
| **qa-tester** | Green | Test generation and execution | Comprehensive test generation, test suite execution, failure diagnosis |
| **ux-auditor** | Magenta | UI/UX evaluation and accessibility | Full UX evaluations, WCAG compliance audits, design system reviews |
### Agent Dispatch Rules

The orchestrator decides whether to route to a **skill** (quick, inline) or an **agent** (deep, multi-file):

- **Skill**: Quick inline tip, single-file fix, one-off question, fast config snippet
- **Agent**: Deep multi-file analysis, codebase-wide audit, structured report, multi-step investigation

### Agent Workflow Pipelines

The **agent-workflows** skill provides pre-built multi-agent pipelines:

| Pipeline | Agents Used | Pattern |
|----------|-------------|---------|
| **PR Review** | code-reviewer + security-scanner | Parallel, then conditional qa-tester |
| **Ship-Ready Check** | code-reviewer + security-scanner + qa-tester | Parallel fan-out, conditional ux-auditor |
| **Full Audit** | All 5 agents | Full parallel fan-out |
| **New Feature** | researcher → code-reviewer → security-scanner → qa-tester | Sequential pipeline |
| **Security Remediation** | security-scanner → fix → code-reviewer → qa-tester → re-scan | Scan-fix-verify loop |
| **Accessibility Overhaul** | ux-auditor → fix → code-reviewer → qa-tester | Audit-fix-verify loop |

### Cross-Agent Handoffs

Each agent produces structured output with a **Handoff Recommendations** section. When an agent identifies issues outside its domain, it recommends which sibling agent to spawn next — enabling automatic agent-to-agent pipelines.
## Skills (34)

### Core

| Skill | Purpose | References |
|-------|---------|------------|
| **orchestrator** | Central brain — routes requests, manages workflow, dispatches agents | routing-guide, long-running-patterns |
| **vibe-coding** | Code generation, scaffolding, feature building across all stacks | scaffolding-templates, coding-patterns, monorepo-patterns |
| **project-scaffolding** | Bootstrap new projects with best-practice structure and config | — |
| **code-quality** | Code review, testing, refactoring, Playwright E2E | review-checklist, testing-strategies, playwright-patterns |
| **deployment** | Multi-platform deployment — Vercel, Cloudflare, Docker, Railway | vercel, cloudflare, docker |
| **research** | Technology evaluation with structured comparison matrices | research-methods |
| **docs-productivity** | Documentation generation — READMEs, API docs, changelogs | doc-templates |
| **codebase-understanding** | Map architecture, trace flows, onboard to unfamiliar projects | — |

### Infrastructure

| Skill | Purpose | References |
|-------|---------|------------|
| **api-design** | REST, GraphQL, tRPC, WebSocket, Stripe integration patterns | api-patterns, stripe-patterns |
| **database** | Drizzle/Prisma migrations, schema design, Supabase, seeding | supabase-patterns |
| **cicd** | GitHub Actions, preview deploys, release automation | — |
| **env-secrets** | Environment variables, secrets safety, multi-env config | — |
| **linting** | Biome, ESLint 9, Prettier, TypeScript strict, pre-commit hooks | — |
| **git-workflow** | Conventional commits, branching strategies, PR best practices | git-commands |
| **monorepo-management** | Turborepo, Nx, pnpm workspaces, package boundaries | — |
### Quality & Design

| Skill | Purpose | References |
|-------|---------|------------|
| **ui-ux** | Design systems, components, WCAG accessibility, Figma-to-code | design-principles, web-design-guidelines, accessibility, figma-to-code |
| **performance** | Core Web Vitals, bundle analysis, caching, rendering optimization | performance-patterns |
| **monitoring** | Error tracking, structured logging, health checks, alerting | monitoring-patterns |
| **security** | OWASP Top 10 vulnerability scanning and hardening | owasp-top-10, security-checklist |
| **structured-logging** | Structured log output with levels, timestamps, JSON formatting | — |

### Intelligence

| Skill | Purpose | References |
|-------|---------|------------|
| **ai-agents** | AI integration, MCP servers, agent architecture, RAG pipelines | mcp-patterns, plugin-building |
| **memory** | Persistent preferences and project context across sessions | memory-patterns |
| **semantic-memory** | Vector-based semantic search over past sessions and project knowledge | — |
| **self-eval** | Auto-evaluation across 5 quality dimensions | eval-criteria |
| **learning** | Closed-loop self-improvement — track failures, detect patterns, adapt | — |
| **context-engine** | Pluggable context management, token budgets, smart windowing | — |

### Operations

| Skill | Purpose | References |
|-------|---------|------------|
| **loop** | Iterative run→check→fix→repeat cycles until CI goes green | loop-patterns |
| **scheduler** | Cron jobs, recurring tasks, scheduled dependency updates | — |
| **daemon-mode** | Persistent background agent for long-running monitoring | — |
| **webhooks** | Event-driven triggers — GitHub events, API calls, Slack messages | — |
| **multi-platform** | Integrate with Slack, Discord, Telegram, WhatsApp, email | — |
### Security & Compliance

| Skill | Purpose | References |
|-------|---------|------------|
| **secrets-management** | Enterprise secrets — Vault, AWS/GCP/Azure managers, SOPS, rotation | — |
| **compliance** | HIPAA, SOC 2, GDPR, PCI DSS code-level enforcement and audit logging | — |

### Orchestration

| Skill | Purpose | References |
|-------|---------|------------|
| **agent-workflows** | Multi-agent pipeline orchestration — PR Review, Ship-Ready, Full Audit | workflow-templates |

See [skills/INDEX.md](skills/INDEX.md) for the full skill catalog with trigger keywords.

## Hooks (5)

VibeCraft includes 5 hooks across 3 lifecycle events for defense-in-depth security and quality automation:

| Hook | Event | What It Does |
|------|-------|-------------|
| **Session Banner** | `SessionStart` | Displays VibeCraft version, lists all skills, agents, and commands |
| **Secret Scanner** | `PreToolUse` (Write/Edit) | Scans code writes for hardcoded API keys, passwords, tokens, and connection strings |
| **Command Safety** | `PreToolUse` (Bash) | Blocks dangerous commands — secret exfiltration, destructive ops, force-push to main |
| **Injection Defense** | `PostToolUse` (WebFetch/WebSearch) | Scans fetched web content for prompt injection attempts |
| **Agent Quality Gate** | `PostToolUse` (Agent) | Checks sub-agent result quality and logs outcomes for self-improvement |

A sixth implicit hook monitors Bash output for test/build failures and suggests engaging the loop engine.
## How It Works

### Orchestrator Routing

The **orchestrator** skill acts as the central brain. When you make a request, it:

1. Detects your intent from the request
2. Checks memory for your preferences and project context
3. Decides whether to route to a **skill** (quick, inline) or spawn an **agent** (deep, multi-file)
4. For complex requests, activates a **workflow pipeline** that coordinates multiple agents
5. After completion, updates memory with new preferences and patterns

### Agent Dispatch

The orchestrator maintains two routing tables — one for skills and one for agents. When the request implies depth ("audit", "thorough review", "full scan"), it spawns an agent. When it implies speed ("quick check", "is this OK?"), it uses a skill. For overlapping domains, clear disambiguation rules prevent confusion.

### MCP-Aware Routing

The orchestrator knows about your connected MCPs and routes through them automatically:

- **Vercel MCP** for deployments and preview URLs
- **Cloudflare MCP** for Workers, D1, R2, KV
- **Atlassian MCP** for Jira tickets and Confluence docs
- **Figma MCP** for design-to-code workflows
- **Context7 MCP** for up-to-date library documentation
- **Google Drive MCP** for accessing existing docs
- And more — Things, AWS, ElevenLabs, Chrome Control, Desktop Commander
### Memory System

VibeCraft maintains a two-tier memory:

- **CLAUDE.md** (hot cache): Your top preferences, active projects, common terms
- **memory/** (deep storage): Full project history, detailed preferences, past decisions

Memory persists across sessions, so Claude gets better the more you use it. The **semantic-memory** skill adds vector-based search, letting you find related context even when the wording differs from what was originally stored.

### Self-Improvement

After each session, the `/improve` command:
- Scores output quality across 5 dimensions
- Identifies what worked well and what didn't
- Updates memory with lessons learned
- Tracks improvement over time via the **learning** skill's closed-loop feedback

### Security Layers

Three hook-based layers provide defense-in-depth:
- **Write/Edit gate**: Prevents committing secrets or unsafe patterns to code
- **Bash gate**: Blocks exfiltration, destructive commands, and encoded payloads
- **Web content gate**: Scans fetched content for prompt injection before processing

The **compliance** skill adds code-level enforcement for HIPAA, SOC 2, GDPR, and PCI DSS. See [SECURITY.md](SECURITY.md) for full details.

## Setup

Install the plugin and start using it immediately. No environment variables or external services required — VibeCraft uses whatever tools and MCPs you already have connected.

For deployment features, ensure you have the relevant CLI tools installed:
- **Vercel**: `npm i -g vercel`
- **Cloudflare**: `npm i -g wrangler`
## Architecture

```
vibecraft/
├── .claude-plugin/plugin.json    # Plugin manifest (v0.8.0)
├── commands/                     # 10 slash commands
│   ├── vibe.md                   # Scaffold and build
│   ├── ship.md                   # Deploy to production
│   ├── review.md                 # Code review
│   ├── research.md               # Technology research
│   ├── plan.md                   # Planning and prioritization
│   ├── secure.md                 # Security scanning
│   ├── test.md                   # Test generation
│   ├── improve.md                # Self-evaluation
│   ├── status.md                 # Project health
│   └── feature.md                # Feature development workflow
├── skills/                       # 34 skills with reference docs
│   ├── orchestrator/             # Central brain + agent dispatch + MCP routing
│   ├── agent-workflows/          # Multi-agent pipeline orchestration
│   ├── vibe-coding/              # Code generation + stack templates
│   ├── project-scaffolding/      # New project bootstrapping
│   ├── code-quality/             # Review, testing, Playwright E2E
│   ├── deployment/               # Vercel, Cloudflare, Docker
│   ├── research/                 # Tech research + decision matrices
│   ├── ui-ux/                    # Design, accessibility, Figma-to-code
│   ├── security/                 # OWASP scanning + hardening
│   ├── compliance/               # HIPAA, SOC 2, GDPR, PCI DSS
│   ├── secrets-management/       # Vault, SOPS, rotation policies
│   ├── memory/                   # Two-tier persistent memory
│   ├── semantic-memory/          # Vector-based semantic search
│   ├── self-eval/                # 5-dimension quality scoring
│   ├── learning/                 # Closed-loop self-improvement
│   ├── context-engine/           # Token budget + context windowing
│   ├── ...and 18 more            # See skills/INDEX.md├── agents/                       # 5 specialized agents with cross-agent handoffs
│   ├── code-reviewer.md          # Deep multi-file code analysis
│   ├── security-scanner.md       # OWASP vulnerability scanning
│   ├── researcher.md             # Multi-source technology research
│   ├── qa-tester.md              # Test generation and execution
│   └── ux-auditor.md             # UX evaluation and accessibility
├── hooks/hooks.json              # 5 hooks: session, secrets, safety, injection, quality
├── GETTING-STARTED.md            # Quick start guide
├── CONTRIBUTING.md               # How to extend VibeCraft
├── SECURITY.md                   # Security policy and compliance
├── CHANGELOG.md                  # Version history
├── LICENSE                       # MIT License
└── README.md                     # This file
```

## Documentation

| Document | Description |
|----------|-------------|
| [README.md](README.md) | Overview, skills, agents, commands, architecture |
| [GETTING-STARTED.md](GETTING-STARTED.md) | Installation and first-project walkthrough |
| [skills/INDEX.md](skills/INDEX.md) | Full catalog of all 34 skills with triggers |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to add skills, hooks, and extend VibeCraft |
| [SECURITY.md](SECURITY.md) | Security hooks, compliance coverage, vulnerability reporting |
| [CHANGELOG.md](CHANGELOG.md) | Version history from v0.1.0 to v0.8.0 |
| [LICENSE](LICENSE) | MIT License |

## License

MIT — see [LICENSE](LICENSE) for the full text.