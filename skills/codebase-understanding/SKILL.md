---
name: codebase-understanding
description: "Understand and map existing codebases, onboard to projects. Use when asked to understand this codebase, how does this work, onboard me, architecture overview, trace data flow, explain this project. Do NOT use for building new projects (use vibe-coding) or code review (use code-quality)."
---

# Codebase Understanding

Rapidly onboard to any existing project. Map the architecture, trace data flows, identify patterns.

## Quick Assessment (< 2 minutes)

1. Read package.json (stack, dependencies, scripts)
2. Read project config files (framework, tooling)
3. List top-level directory structure
4. Read README.md (purpose and setup)
5. Check git log (recent activity)

### Stack Detection
```bash
cat package.json | grep -E '"(next|react|vue|svelte|hono|express)"'
[ -f "next.config.mjs" ] && echo "Framework: Next.js"
[ -f "wrangler.toml" ] && echo "Platform: Cloudflare Workers"
[ -f "vercel.json" ] && echo "Platform: Vercel"
[ -f "turbo.json" ] && echo "Monorepo: Turborepo"
```

## Architecture Mapping

### Directory Patterns
- `app/`: Next.js App Router
- `pages/` + `api/`: Next.js Pages Router
- `src/routes/`: SvelteKit / Remix
- `src/workers/`: Cloudflare Workers
- `apps/` + `packages/`: Monorepo
- `src/controllers/`: MVC/Service pattern

### Key Files to Read First
- `package.json`: Stack, scripts, dependencies
- `tsconfig.json`: Path aliases, strictness
- `.env.example`: External services
- Main layout: Providers, global state
- Database schema: Data model, relationships
- Auth config: Auth flow, providers
- API routes (2-3): Patterns, error handling

## Data Flow Tracing

### Trace Feature End-to-End
Given: "How does user signup work?"
1. Find signup form → grep "signup"
2. Trace submission → onSubmit handler
3. Find API endpoint
4. Trace to database → what gets created
5. Check side effects → emails, webhooks
6. Verify error handling

### Search Patterns
```bash
grep -r "export.*function" --include="*.ts" -l
find . -path "*/api/*" -name "route.ts"
grep -r "from(users)" --include="*.ts" -l
grep -r '"use server"' --include="*.ts" -l
grep -r "auth\|session" --include="*.ts" -l
```

## Generate Architecture Doc

Save to memory after understanding:
- Stack (framework, DB, auth, deployment)
- Directory structure (with annotations)
- Data model (key entities)
- Key flows (auth, main feature)
- API surface (routes/endpoints)
- Known patterns (conventions)
- Tech debt (gotchas, workarounds)

## Onboarding Checklist

- [ ] Run stack detection
- [ ] Read README + docs
- [ ] Read database schema
- [ ] Read auth setup
- [ ] Trace 2-3 key features
- [ ] Read test setup
- [ ] Read CI/CD config
- [ ] Check CLAUDE.md or memory/
- [ ] Run project locally
- [ ] Generate architecture doc

## Memory Integration

After understanding a codebase:
1. Update CLAUDE.md with stack, patterns
2. Save architecture to memory/projects/{project}.md
3. Note non-obvious patterns
4. Track tech debt discovered

Read `references/onboarding-guide.md` for detailed onboarding workflows.
