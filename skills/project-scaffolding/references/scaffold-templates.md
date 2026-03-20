# Scaffold Templates Reference

Complete project templates for every supported stack. Each template includes full directory structure, config files, and entry points.

## Next.js (App Router) — Fullstack Web App

### Directory Structure
```
my-app/
├── src/
│   ├── app/
│   │   ├── layout.tsx          # Root layout with metadata
│   │   ├── page.tsx            # Home page
│   │   ├── globals.css         # Tailwind imports + global styles
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── dashboard/
│   │   │   ├── layout.tsx      # Dashboard layout with sidebar
│   │   │   └── page.tsx
│   │   └── api/
│   │       └── health/route.ts # Health check endpoint
│   ├── components/
│   │   ├── ui/                 # Reusable primitives (Button, Input, Card)
│   │   └── layout/             # Header, Footer, Sidebar
│   ├── lib/
│   │   ├── utils.ts            # Utility functions
│   │   └── cn.ts               # clsx + tailwind-merge helper
│   └── types/
│       └── index.ts            # Shared TypeScript types
├── public/
│   └── favicon.ico
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── postcss.config.mjs
├── package.json
├── .eslintrc.json
├── .prettierrc
├── .gitignore
├── .env.example
└── README.md
```

### Key Config: tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### Key Config: package.json scripts
```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit"
  }
}
```

## React + Vite — SPA

### Directory Structure
```
my-spa/
├── src/
│   ├── main.tsx                # Entry point with React.StrictMode
│   ├── App.tsx                 # Root component with router
│   ├── index.css               # Tailwind imports
│   ├── routes/
│   │   ├── Home.tsx
│   │   └── About.tsx
│   ├── components/
│   │   └── ui/
│   ├── hooks/
│   │   └── useLocalStorage.ts
│   ├── lib/
│   │   └── api.ts              # API client (fetch wrapper)
│   ├── stores/
│   │   └── useAppStore.ts      # Zustand store
│   └── types/
│       └── index.ts
├── vite.config.ts
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── tailwind.config.ts
├── postcss.config.js
├── package.json
├── index.html
├── .eslintrc.json
├── .prettierrc
└── .gitignore
```

## Hono API — Cloudflare Workers

### Directory Structure
```
my-api/
├── src/
│   ├── index.ts                # Entry point with Hono app
│   ├── routes/
│   │   ├── health.ts           # GET /health
│   │   └── users.ts            # CRUD /users
│   ├── middleware/
│   │   ├── auth.ts             # JWT/API key validation
│   │   └── cors.ts             # CORS config
│   ├── lib/
│   │   ├── db.ts               # D1 database client
│   │   └── errors.ts           # Error classes
│   └── types/
│       └── env.ts              # Cloudflare bindings type
├── wrangler.toml
├── tsconfig.json
├── package.json
├── vitest.config.ts
├── .dev.vars                   # Local secrets (gitignored)
├── .gitignore
└── README.md
```

### Key Config: wrangler.toml
```toml
name = "my-api"
main = "src/index.ts"
compatibility_date = "2024-12-01"

[vars]
ENVIRONMENT = "development"

[[d1_databases]]
binding = "DB"
database_name = "my-api-db"
database_id = "placeholder-replace-me"
```

## FastAPI — Python API

### Directory Structure
```
my-api/
├── src/
│   └── my_api/
│       ├── __init__.py
│       ├── main.py             # FastAPI app with lifespan
│       ├── routers/
│       │   ├── __init__.py
│       │   ├── health.py
│       │   └── users.py
│       ├── models/
│       │   ├── __init__.py
│       │   └── user.py         # Pydantic models
│       ├── services/
│       │   └── user_service.py
│       ├── db/
│       │   ├── __init__.py
│       │   └── session.py      # SQLAlchemy session
│       └── config.py           # Pydantic Settings
├── tests/
│   ├── conftest.py
│   └── test_health.py
├── pyproject.toml
├── Dockerfile
├── .env.example
├── .gitignore
└── README.md
```

### Key Config: pyproject.toml
```toml
[project]
name = "my-api"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn[standard]>=0.32.0",
    "pydantic-settings>=2.6.0",
]

[project.optional-dependencies]
dev = ["pytest>=8.0", "httpx>=0.27", "ruff>=0.8"]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "SIM"]
```

## Go API — Chi Router

### Directory Structure
```
my-api/
├── cmd/
│   └── server/
│       └── main.go             # Entry point
├── internal/
│   ├── handler/
│   │   ├── health.go
│   │   └── users.go
│   ├── middleware/
│   │   └── auth.go
│   ├── model/
│   │   └── user.go
│   └── service/
│       └── user.go
├── go.mod
├── go.sum
├── Dockerfile
├── Makefile
├── .gitignore
└── README.md
```

## Node.js CLI — Commander

### Directory Structure
```
my-cli/
├── src/
│   ├── index.ts                # Entry with Commander program
│   ├── commands/
│   │   ├── init.ts
│   │   └── generate.ts
│   ├── lib/
│   │   ├── config.ts           # Config file loading
│   │   └── logger.ts           # Chalk-based logger
│   └── types/
│       └── index.ts
├── tsconfig.json
├── tsup.config.ts              # Bundle to single CJS file
├── package.json                # bin field pointing to dist
├── .eslintrc.json
├── .gitignore
└── README.md
```

## Library/Package — Dual CJS/ESM

### Directory Structure
```
my-lib/
├── src/
│   ├── index.ts                # Public API barrel export
│   ├── core.ts
│   └── utils.ts
├── tests/
│   └── core.test.ts
├── tsconfig.json
├── tsup.config.ts              # Dual format output
├── vitest.config.ts
├── package.json
├── .changeset/
│   └── config.json             # Changesets for versioning
├── .eslintrc.json
├── .prettierrc
├── .gitignore
└── README.md
```

### Key Config: tsup.config.ts
```ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["cjs", "esm"],
  dts: true,
  splitting: false,
  sourcemap: true,
  clean: true,
});
```

### Key Config: package.json exports
```json
{
  "name": "my-lib",
  "version": "0.0.1",
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.cjs",
      "types": "./dist/index.d.ts"
    }
  },
  "files": ["dist"]
}
```

## Docker — Multi-Stage Build (Node.js)

```dockerfile
FROM node:20-alpine AS base
RUN corepack enable && corepack prepare pnpm@latest --activate

FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

FROM base AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm build

FROM base AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY package.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## Common .gitignore (Node.js)

```
node_modules/
dist/
.next/
.turbo/
.env
.env.local
.env.*.local
*.tsbuildinfo
.DS_Store
coverage/
```

## Common .editorconfig

```ini
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

[*.py]
indent_size = 4

[*.go]
indent_style = tab
```