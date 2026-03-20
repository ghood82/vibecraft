# Advanced Monorepo Patterns

## Turborepo Setup

```
my-monorepo/
├── apps/
│   ├── web/              # Next.js frontend
│   │   ├── package.json
│   │   └── CLAUDE.md     # Web-specific context
│   ├── api/              # Hono API
│   │   ├── package.json
│   │   └── CLAUDE.md     # API-specific context
│   └── docs/             # Documentation site
│       └── package.json
├── packages/
│   ├── ui/               # Shared component library
│   │   ├── src/
│   │   └── package.json
│   ├── db/               # Shared database schema + client
│   │   ├── src/
│   │   │   ├── schema.ts
│   │   │   ├── client.ts
│   │   │   └── migrations/
│   │   └── package.json
│   ├── config-eslint/    # Shared ESLint config
│   │   └── package.json
│   ├── config-ts/        # Shared TypeScript config
│   │   └── package.json
│   └── shared/           # Shared types, utils, constants
│       ├── src/
│       └── package.json
├── turbo.json
├── package.json
├── pnpm-workspace.yaml
└── CLAUDE.md             # Root context: architecture, conventions
```

### turbo.json
```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "db:migrate": {
      "cache": false
    }
  }
}
```

### pnpm-workspace.yaml
```yaml
packages:
  - "apps/*"
  - "packages/*"
```

### Root package.json
```json
{
  "private": true,
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "lint": "turbo lint",
    "test": "turbo test",
    "db:migrate": "turbo db:migrate --filter=@repo/db",
    "clean": "turbo clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^2.0.0"
  }
}
```

## Nested CLAUDE.md Pattern

Each package/app gets its own CLAUDE.md for context that's specific to that part of the monorepo:

```markdown
<!-- apps/web/CLAUDE.md -->
# Web App Context

## Stack
- Next.js 15 App Router
- Tailwind CSS + shadcn/ui
- TanStack Query for data fetching

## Conventions
- All pages in app/(dashboard)/ are behind auth
- Use `@repo/ui` for shared components
- Use `@repo/db` for database access in server components
- API calls go through `/api/` routes or server actions

## Current Work
- Migrating from Pages Router to App Router
- Dashboard redesign in progress
```

```markdown
<!-- packages/db/CLAUDE.md -->
# Database Package Context

## Stack
- Drizzle ORM + PostgreSQL (Neon)
- Schema in src/schema.ts
- Migrations in drizzle/

## Conventions
- All tables use UUID primary keys
- Timestamps: createdAt + updatedAt on every table
- Soft delete: use deletedAt column
- Export types from src/types.ts for consumer packages
```

## Shared Package Patterns

### Internal Package Setup
```json
// packages/ui/package.json
{
  "name": "@repo/ui",
  "version": "0.0.0",
  "private": true,
  "exports": {
    ".": "./src/index.ts",
    "./button": "./src/button.tsx",
    "./card": "./src/card.tsx"
  },
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "lint": "biome check src/"
  },
  "devDependencies": {
    "@repo/config-ts": "workspace:*",
    "tsup": "^8.0.0"
  }
}
```

### Cross-Package Imports
```typescript
// apps/web/app/page.tsx
import { Button } from "@repo/ui/button";
import { db } from "@repo/db";
import { users } from "@repo/db/schema";
import type { User } from "@repo/shared/types";
```

## Changesets (Versioning & Publishing)

```bash
npm install -D @changesets/cli
npx changeset init
```

```json
// .changeset/config.json
{
  "$schema": "https://unpkg.com/@changesets/config@3.0.0/schema.json",
  "changelog": ["@changesets/changelog-github", { "repo": "user/repo" }],
  "commit": false,
  "fixed": [],
  "linked": [["@repo/ui", "@repo/shared"]],
  "access": "restricted",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": ["@repo/web", "@repo/api"]
}
```

Workflow:
```bash
# When you make a change to a package
npx changeset
# Select affected packages, write change summary
# Creates .changeset/random-name.md

# When ready to release
npx changeset version   # Bumps versions, updates changelogs
npx changeset publish   # Publishes to npm (if public)
```

## Dependency Management

### Internal Dependencies
```json
// apps/web/package.json
{
  "dependencies": {
    "@repo/ui": "workspace:*",
    "@repo/db": "workspace:*",
    "@repo/shared": "workspace:*"
  }
}
```

### Shared External Dependencies
```bash
# Install a dependency in a specific package
pnpm add zod --filter @repo/shared

# Install a dev dependency at root (shared tool)
pnpm add -D vitest -w

# Install across multiple packages
pnpm add react --filter "@repo/ui" --filter "@repo/web"
```

### Filtering Turbo Commands
```bash
# Build only the web app and its dependencies
turbo build --filter=@repo/web

# Run tests only in packages that changed since main
turbo test --filter="...[main]"

# Run dev for web + api simultaneously
turbo dev --filter=@repo/web --filter=@repo/api
```

## CI for Monorepos

```yaml
# .github/workflows/ci.yml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.filter.outputs.changes }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            web: 'apps/web/**'
            api: 'apps/api/**'
            ui: 'packages/ui/**'
            db: 'packages/db/**'

  build:
    needs: changes
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo build --filter="...[HEAD~1]"
```

## Common Patterns

### Shared Environment Config
```typescript
// packages/config-env/src/index.ts
import { z } from "zod";

export const sharedEnvSchema = {
  DATABASE_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
};

// apps/web/src/env.ts
import { createEnv } from "@t3-oss/env-nextjs";
import { sharedEnvSchema } from "@repo/config-env";

export const env = createEnv({
  server: {
    ...sharedEnvSchema,
    AUTH_SECRET: z.string(),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
  },
  runtimeEnv: { /* ... */ },
});
```

### Shared Database Access
```typescript
// packages/db/src/client.ts
import { drizzle } from "drizzle-orm/neon-http";
import { neon } from "@neondatabase/serverless";
import * as schema from "./schema";

export function createDb(databaseUrl: string) {
  const sql = neon(databaseUrl);
  return drizzle(sql, { schema });
}

export type Database = ReturnType<typeof createDb>;

// apps/web/src/lib/db.ts
import { createDb } from "@repo/db";
import { env } from "@/env";
export const db = createDb(env.DATABASE_URL);

// apps/api/src/lib/db.ts
import { createDb } from "@repo/db";
export const db = createDb(process.env.DATABASE_URL!);
```

## Gotchas

- **TypeScript paths**: Each app needs its own `tsconfig.json` with correct `paths` to workspace packages
- **Build order**: `turbo.json` `dependsOn: ["^build"]` ensures packages build before apps
- **Dev mode**: Internal packages with `"exports"` pointing to `src/` work without building in dev. For production, build first with tsup.
- **Hoisted dependencies**: pnpm doesn't hoist by default — if a package needs `react`, add it to that package's `package.json`
- **Deployment**: Vercel auto-detects monorepos. Set the "Root Directory" to `apps/web` for the web app.
