---
name: monorepo-management
description: "Set up and manage monorepos — Turborepo, Nx, pnpm workspaces, package boundaries, selective builds, shared libraries. Use when asked about monorepo, workspace, turbo, nx, lerna, shared packages, internal libraries, multi-package. Do NOT use for single-project scaffolding (use project-scaffolding) or CI pipelines (use cicd)."
---

# Monorepo Management

Set up, structure, and optimize monorepos. Covers tooling selection, workspace configuration, package boundaries, build caching, and CI integration.

## Core Principles

1. **Boundaries matter** — Every package has a clear public API. No reaching into internals.
2. **Build only what changed** — Selective execution via dependency graph. Never rebuild everything.
3. **Shared code, not copied code** — Internal packages for common utilities, UI, config, types.
4. **One lockfile** — Single source of truth for dependencies at the root.
5. **Independent when possible** — Packages should build, test, and lint in isolation.

## Monorepo Setup Workflow

1. **Assess scope** — How many apps? How many shared packages? What stacks?
2. **Choose tooling** — Match to project needs (see Tooling Selection below)
3. **Configure workspaces** — Root package.json + workspace globs
4. **Create package structure** — apps/, packages/, tooling/ directories
5. **Set up shared config** — TypeScript, ESLint, Prettier, Tailwind as internal packages
6. **Configure build orchestration** — turbo.json or nx.json with task dependencies
7. **Wire CI** — Affected-only builds, remote caching, parallel jobs
8. **Document conventions** — Package naming, dependency rules, contribution guide

## Tooling Selection

| Tool | Best For | Key Strength |
|------|----------|-------------|
| **Turborepo** | JS/TS monorepos, simple setup | Zero-config caching, fast adoption |
| **Nx** | Large orgs, polyglot repos | Generators, module boundaries, plugins |
| **pnpm workspaces** | Lightweight, no build orchestrator needed | Strict dependency isolation, fast installs |
| **Lerna** (legacy) | Existing Lerna repos needing maintenance | Publishing workflows (use Changesets instead for new projects) |

### Decision Logic
```
Small team, JS/TS only, want simplicity? → Turborepo + pnpm
Large org, need generators and constraints? → Nx
Just need workspaces, no build orchestration? → pnpm workspaces alone
Publishing many npm packages? → Turborepo + Changesets
Polyglot (JS + Go + Python)? → Nx or Bazel
```

## Standard Directory Structure

```
monorepo/
├── apps/
│   ├── web/                 # Next.js frontend
│   ├── api/                 # Hono/Express API
│   └── mobile/              # React Native
├── packages/
│   ├── ui/                  # Shared component library
│   ├── utils/               # Shared utilities
│   ├── types/               # Shared TypeScript types
│   ├── db/                  # Database schema + client
│   └── config/              # Shared ESLint, TS, Tailwind configs
├── tooling/
│   ├── eslint/              # ESLint shared config package
│   ├── typescript/          # tsconfig base files
│   └── tailwind/            # Tailwind preset
├── turbo.json               # Build orchestration
├── pnpm-workspace.yaml      # Workspace definition
├── package.json             # Root scripts + devDependencies
└── .github/workflows/ci.yml # CI with affected filtering
```

## Package Boundaries

### Rules
- Apps import from packages, never from other apps
- Packages declare explicit `exports` in package.json — no deep imports
- Shared types live in `@repo/types`, not duplicated across packages
- Database client is a package (`@repo/db`), not embedded in an app
- Config packages (`@repo/eslint-config`) extend, apps consume

### Internal Package Template
Every internal package needs:
- `package.json` with `name`, `exports`, `types`, `scripts`
- `tsconfig.json` extending the shared base
- `src/index.ts` as the public API barrel export
- Build step (tsup) OR `"main": "./src/index.ts"` for unbundled internal use

## Build & Cache Strategy

### Turborepo Task Dependencies
```jsonc
// turbo.json
{
  "tasks": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**", ".next/**"] },
    "test": { "dependsOn": ["build"] },
    "lint": {},
    "dev": { "cache": false, "persistent": true }
  }
}
```

### Caching
- **Local cache**: Automatic with Turborepo/Nx (`.turbo/` or `.nx/`)
- **Remote cache**: Vercel Remote Cache (Turbo) or Nx Cloud for team sharing
- **CI cache**: Cache node_modules and turbo cache between runs
- **Cache inputs**: Include source files, configs, lockfile — exclude tests from build cache

## Versioning Strategies

| Strategy | When to Use |
|----------|-------------|
| **Fixed/unified** | All packages share one version (e.g., design system) |
| **Independent** | Packages evolve at their own pace (e.g., utility libs) |
| **Changesets** | Automated changelog + version bumps + npm publish |

## CI for Monorepos

### Affected-Only Builds
```yaml
# Only run CI for packages that changed
- name: Build affected
  run: pnpm turbo build --filter=...[origin/main]
```

### Key CI Patterns
- Use `--filter` (Turbo) or `--affected` (Nx) to skip unchanged packages
- Cache `node_modules` and `.turbo` between runs
- Run lint/test/build as separate jobs for parallelism
- Deploy apps independently — each app has its own deploy job
- Use path filters on GitHub Actions to skip CI entirely when only docs change

## Common Mistakes

- **God package**: One `shared` package that everything depends on (split it up)
- **Circular deps**: Package A imports B which imports A (restructure boundaries)
- **No build order**: Packages build before their dependencies (use `dependsOn: ["^build"]`)
- **Root dependencies**: Installing app-specific deps at root (install in the app's package.json)
- **Missing exports**: Packages without explicit `exports` field (breaks encapsulation)

## Memory Integration

After monorepo setup, persist to memory:
- Tool choice and reasoning ("chose Turborepo for simplicity, team is small")
- Package naming convention ("@repo/ prefix for internal packages")
- Workspace structure ("apps/ for deployables, packages/ for shared code")
- Versioning strategy ("independent versions with Changesets")

Read `references/monorepo-patterns.md` for detailed configs, templates, and migration guides.