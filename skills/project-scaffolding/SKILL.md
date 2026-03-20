---
name: project-scaffolding
description: "Scaffold new projects from scratch with best-practice structure, config, and boilerplate. Use when asked to init, bootstrap, create new project, start fresh, generate starter, new app, new API, new package. Do NOT use for adding features to existing projects (use vibe-coding) or monorepo setup (use monorepo-management)."
---

# Project Scaffolding

Zero-to-running project generation for any stack. Detect intent, pick the right template, generate everything, verify it runs.

## Core Principles

1. **One command to running** — After scaffolding, `npm run dev` (or equivalent) must work
2. **Convention-first** — Use each ecosystem's canonical structure (App Router for Next.js, src/ layout for Vite, etc.)
3. **TypeScript by default** — Unless the user explicitly asks for JavaScript or Python
4. **Config complete** — Every project gets tsconfig, linting, formatting, .gitignore, and editor config from day one
5. **No dead code** — Every generated file has a purpose. No placeholder comments or TODO stubs.

## Scaffolding Workflow

1. **Detect intent** — What type of project? (web app, API, CLI, library, extension, fullstack)
2. **Check memory** — Read CLAUDE.md and memory/ for stack preferences, team conventions
3. **Select template** — Match to a template from `references/scaffold-templates.md`
4. **Generate structure** — Directories, config files, source files, scripts
5. **Wire dependencies** — package.json / pyproject.toml / go.mod / Cargo.toml with pinned versions
6. **Create entry point** — A working hello-world that proves the setup compiles and runs
7. **Add DX tooling** — Linting, formatting, pre-commit hooks, editor config
8. **Verify** — Run build or dev server to confirm zero errors

## Supported Stacks

### Frontend
- **Next.js (App Router)** — React 19, server components, Tailwind, TypeScript
- **React + Vite** — Fast HMR, React Router, Zustand/Jotai
- **Astro** — Content sites, islands architecture
- **SvelteKit** — Full-stack Svelte

### Backend
- **Hono** — Lightweight, runs on Workers/Node/Deno/Bun
- **Express** — Classic Node.js, TypeScript-first
- **FastAPI** — Python async API with Pydantic
- **Django** — Python batteries-included
- **Go (net/http + Chi)** — Minimal Go API
- **Rust (Axum)** — High-performance Rust API

### Fullstack
- **Next.js + Drizzle/Prisma** — App Router + database + auth
- **T3 Stack** — Next.js + tRPC + Prisma + NextAuth + Tailwind
- **SvelteKit + Drizzle** — Fullstack Svelte

### Other
- **CLI (Node)** — Commander/CAC + TypeScript
- **CLI (Go)** — Cobra + Viper
- **Library/Package** — Dual CJS/ESM, tsup bundler, Changesets for versioning
- **Browser Extension** — Manifest V3, Vite, React/Svelte popup
- **MCP Server** — Model Context Protocol server (TypeScript or Python)

## Config Generation

Every project gets these configs tailored to its stack:

| Config | Purpose |
|--------|---------|
| `tsconfig.json` | Strict TypeScript (or language-equivalent) |
| `.eslintrc` / `eslint.config.js` | Flat config, stack-appropriate rules |
| `.prettierrc` | Consistent formatting |
| `.gitignore` | Language + framework specific |
| `.editorconfig` | Cross-editor consistency |
| `Dockerfile` | Multi-stage build (when requested) |
| `.env.example` | Document required env vars (never actual secrets) |
| `.github/workflows/ci.yml` | Basic lint + test + build pipeline |

## Template Selection Logic

```
Is it a web app with SSR/SSG? → Next.js or SvelteKit or Astro
Is it a SPA? → React + Vite
Is it an API only? → Hono (Workers) or Express (Node) or FastAPI (Python)
Is it fullstack? → Next.js + DB or T3 Stack
Is it a library? → tsup + Changesets
Is it a CLI? → Commander (Node) or Cobra (Go)
Is it a browser extension? → Manifest V3 + Vite
Is it an MCP server? → @modelcontextprotocol/sdk
Does user have preferences in memory? → Override defaults with those
```

## Post-Scaffold Checklist

After generating a project, verify:
- [ ] `npm install` (or equivalent) succeeds with no warnings
- [ ] `npm run dev` starts without errors
- [ ] `npm run build` produces output
- [ ] `npm run lint` passes clean
- [ ] TypeScript has zero errors
- [ ] .gitignore covers node_modules, .env, build output
- [ ] README.md has setup instructions

## Memory Integration

After scaffolding, persist to memory:
- Stack choice and reasoning ("chose Hono over Express for Workers compat")
- Config preferences ("user wants tabs, not spaces" / "single quotes")
- Directory structure pattern ("feature-based, not type-based")
- Package manager preference ("user uses pnpm")

Read `references/scaffold-templates.md` for complete project templates with every file.