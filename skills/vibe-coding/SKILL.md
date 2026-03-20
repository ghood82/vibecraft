---
name: vibe-coding
description: "Scaffold projects, generate code, build features fast. Use when asked to build, create, scaffold, new project, prototype, add feature, implement, code. Do NOT use for code review (use code-quality) or deployment (use deployment)."
---

# Vibe Coding Engine

Build fast, ship fast, iterate fast. This skill handles everything from scaffolding a greenfield project to adding features to an existing codebase.

## Core Principles

1. **Detect, don't ask** — Read config files, package.json, existing code to understand the stack before asking questions
2. **Convention over configuration** — Use sensible defaults that match the ecosystem (e.g., App Router for Next.js, not Pages)
3. **Type everything** — TypeScript by default unless the user explicitly wants JavaScript
4. **Ship incrementally** — Get something working first, then improve. Never spend 20 minutes in silence.
5. **Clean from the start** — Proper file structure, named exports, error handling. No "clean it up later."

## Scaffolding Workflow

When creating a new project:

1. **Detect intent** — What are they building? (web app, API, CLI, extension, etc.)
2. **Check memory** — Do they have stack preferences in CLAUDE.md? Use those.
3. **Select template** — Match to the right scaffolding template from `references/scaffolding-templates.md`
4. **Generate structure** — Create directories, config files, initial code
5. **Wire dependencies** — package.json, tsconfig, tailwind.config, etc.
6. **Create entry point** — A working "hello world" that proves the setup works
7. **Verify** — Run the dev server or build to confirm everything works

Read `references/scaffolding-templates.md` for complete project templates.

## Feature Building Workflow

When adding features to an existing project:

1. **Read the codebase** — Understand existing patterns, file structure, naming conventions
2. **Match the style** — New code should look like it was written by the same person
3. **Generate the feature** — Components, routes, API endpoints, database schema
4. **Wire it up** — Imports, routing, navigation, state management
5. **Handle edges** — Loading states, error states, empty states, validation
6. **Type it** — Full TypeScript types, Zod schemas for runtime validation

## Code Generation Standards

### File Organization
- One component per file, named exports
- Colocate related files (component + styles + tests + types)
- Barrel exports only at module boundaries, not everywhere
- Keep files under 300 lines; split when they grow

### Naming
- Components: PascalCase (`UserProfile.tsx`)
- Utilities: camelCase (`formatDate.ts`)
- Constants: SCREAMING_SNAKE for true constants, camelCase for config
- Types/Interfaces: PascalCase, no `I` prefix

### Patterns
- Prefer composition over inheritance
- Use custom hooks to extract logic from components
- Server components by default in Next.js (client only when needed)
- Dependency injection for testability
- Early returns over deep nesting

### Error Handling
- Never swallow errors silently
- Use typed error classes for domain errors
- Return Result types for expected failures, throw for unexpected
- User-facing errors need human-readable messages
- Log enough context to debug without reproducing

Read `references/coding-patterns.md` for detailed patterns with code examples.

## Stack-Specific Guidance

### Next.js (App Router)
- App Router with server components by default
- Server Actions for mutations
- Route groups for layout organization
- Middleware for auth/redirects
- Image optimization with next/image

### Cloudflare Workers
- Hono for routing (lightweight, Workers-native)
- D1 for SQL, KV for cache, R2 for files
- Wrangler for local dev and deployment
- Keep cold start in mind — minimize imports

### React + Vite
- Vite for speed, React Router for routing
- Zustand or Jotai for state (not Redux unless complex)
- TanStack Query for server state

### Node.js APIs
- Hono or Express depending on preference
- Zod for input validation
- Drizzle or Prisma for database
- Structured logging from day one

### Python
- FastAPI for APIs, Pydantic for validation
- Poetry or uv for dependency management
- Type hints everywhere

## Rapid Prototyping Mode

When the user says "quick", "prototype", "just get something working", or "MVP":

- Skip tests (mark as follow-up items)
- Use inline styles or basic Tailwind
- Hardcode what can be configured later
- Single file is fine for small prototypes
- Focus on the core interaction, not polish
- BUT: still use TypeScript, still handle errors, still structure sanely

## Memory Integration

After generating code, note in memory:
- Stack choices made ("chose Drizzle over Prisma for this project")
- Patterns used ("user prefers named exports")
- Config decisions ("tailwind with custom color palette")
- File structure patterns ("components organized by feature, not type")
