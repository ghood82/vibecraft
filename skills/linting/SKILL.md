---
name: linting
description: "Linting, formatting, code style (Biome, ESLint, Prettier, TypeScript strict). Use for lint setup. Do NOT use for code review, testing, or CI/CD pipelines."
---

# Linting & Formatting

Automate code style. Covers Biome (recommended), ESLint 9 flat config, Prettier, TypeScript strict, and pre-commit hooks.

## Which Setup to Choose

| Project Type | Recommendation |
|-------------|---------------|
| New project, simple needs | **Biome** (fastest, least config) |
| React/Next.js with a11y needs | **ESLint + Prettier** (more plugins) |
| Monorepo | **Biome** with per-package overrides |
| Legacy project | Add **Prettier** first, then ESLint gradually |
| Team project | Whatever the team already uses |

## Biome (Recommended)

```bash
npm install --save-dev --save-exact @biomejs/biome
npx @biomejs/biome init
```

All-in-one linter + formatter. Written in Rust. Replaces ESLint + Prettier.

```json
// package.json scripts
{
  "lint": "biome check .",
  "lint:fix": "biome check --write .",
  "format": "biome format --write ."
}
```

## ESLint 9 (Flat Config)

For projects needing specific ESLint plugins (React hooks, accessibility):

```bash
npm install -D eslint @eslint/js typescript-eslint eslint-plugin-react-hooks eslint-plugin-jsx-a11y
```

Uses `eslint.config.ts` (flat config) — no more `.eslintrc`.

## Pre-Commit Hooks (Husky + lint-staged)

```bash
npm install -D husky lint-staged
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["biome check --write", "biome format --write"],
    "*.{json,md,yml,yaml}": ["prettier --write"]
  }
}
```

## TypeScript Strict Mode

Key flags in `tsconfig.json`:
- `"strict": true` — enables all strict checks
- `"noUncheckedIndexedAccess": true` — `arr[0]` is `T | undefined`
- `"noImplicitReturns": true`
- `"forceConsistentCasingInFileNames": true`
- `"moduleResolution": "bundler"` — modern resolution

## Commitlint (Conventional Commits)

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

Enforces `feat:`, `fix:`, `docs:`, etc. prefixes on commit messages.

## Setup Checklist

- [ ] Linter configured (Biome or ESLint)
- [ ] Formatter configured (Biome or Prettier)
- [ ] TypeScript strict mode enabled
- [ ] Pre-commit hooks with lint-staged
- [ ] VS Code settings committed
- [ ] CI runs lint check
- [ ] Import sorting configured
- [ ] Tailwind class sorting (prettier-plugin-tailwindcss)

## Reference Docs

Read `references/linting-configs.md` for full Biome config, ESLint flat config, Prettier config, tsconfig, VS Code settings, and commitlint setup.
