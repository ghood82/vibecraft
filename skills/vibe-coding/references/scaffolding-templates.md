# Scaffolding Templates

Complete project templates for common stacks. Each includes directory structure, key config files, and setup commands.

## Next.js App Router

```
my-app/
├── src/
│   ├── app/
│   │   ├── layout.tsx           # Root layout
│   │   ├── page.tsx             # Home page
│   │   ├── globals.css          # Global styles
│   │   ├── (auth)/              # Auth route group
│   │   │   ├── login/page.tsx
│   │   │   └── signup/page.tsx
│   │   ├── (dashboard)/         # Dashboard route group
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   └── api/                 # API routes (if needed)
│   ├── components/
│   │   ├── ui/                  # Reusable UI components
│   │   └── [feature]/           # Feature-specific components
│   ├── lib/
│   │   ├── db/
│   │   │   ├── schema.ts        # Drizzle schema
│   │   │   ├── index.ts         # DB client
│   │   │   └── migrations/
│   │   ├── auth.ts              # Auth helpers
│   │   └── utils.ts             # General utilities
│   ├── hooks/                   # Custom React hooks
│   └── types/                   # Shared TypeScript types
├── public/
├── drizzle.config.ts
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── .env.local
```

**Setup:**
```bash
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd my-app
npm install drizzle-orm better-sqlite3
npm install -D drizzle-kit @types/better-sqlite3
```

## Cloudflare Workers

```
my-worker/
├── src/
│   ├── index.ts                 # Entry point with Hono app
│   ├── routes/
│   │   ├── api.ts               # API routes
│   │   └── auth.ts              # Auth routes
│   ├── middleware/
│   │   ├── cors.ts
│   │   └── auth.ts
│   ├── db/
│   │   ├── schema.sql           # D1 schema
│   │   └── queries.ts           # Type-safe queries
│   ├── services/                # Business logic
│   └── types.ts                 # Bindings and types
├── migrations/                  # D1 migrations
├── wrangler.toml
├── tsconfig.json
├── package.json
└── vitest.config.ts
```

**wrangler.toml:**
```toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[[d1_databases]]
binding = "DB"
database_name = "my-db"
database_id = "xxx"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "my-bucket"

[[kv_namespaces]]
binding = "CACHE"
id = "xxx"
```

**Setup:**
```bash
npm create cloudflare@latest my-worker -- --type worker-typescript
cd my-worker
npm install hono
```

## React + Vite

```
my-app/
├── src/
│   ├── main.tsx                 # Entry point
│   ├── App.tsx                  # Root component
│   ├── routes/                  # Page components
│   │   ├── index.tsx
│   │   └── about.tsx
│   ├── components/
│   │   ├── ui/                  # Reusable components
│   │   └── layout/              # Layout components
│   ├── hooks/
│   ├── lib/
│   │   ├── api.ts               # API client
│   │   └── utils.ts
│   ├── stores/                  # Zustand stores
│   ├── types/
│   └── index.css
├── public/
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.ts
└── package.json
```

**Setup:**
```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install react-router-dom zustand @tanstack/react-query
npx tailwindcss init -p
```

## Node.js API (Hono)

```
my-api/
├── src/
│   ├── index.ts                 # Entry + Hono app
│   ├── routes/
│   │   ├── users.ts
│   │   └── health.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── cors.ts
│   │   └── logger.ts
│   ├── db/
│   │   ├── schema.ts            # Drizzle schema
│   │   ├── index.ts             # Connection
│   │   └── migrations/
│   ├── services/                # Business logic
│   ├── validators/              # Zod schemas
│   └── types/
├── tests/
├── drizzle.config.ts
├── tsconfig.json
├── package.json
└── .env
```

**Setup:**
```bash
mkdir my-api && cd my-api
npm init -y
npm install hono @hono/node-server drizzle-orm better-sqlite3 zod
npm install -D typescript @types/node @types/better-sqlite3 drizzle-kit tsx vitest
```

## Python FastAPI

```
my-api/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI app
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── health.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py              # Pydantic models
│   ├── db/
│   │   ├── __init__.py
│   │   ├── database.py          # Connection
│   │   └── models.py            # SQLAlchemy models
│   ├── services/
│   └── middleware/
├── tests/
├── pyproject.toml
├── .env
└── alembic/                     # Migrations
```

**Setup:**
```bash
mkdir my-api && cd my-api
uv init
uv add fastapi uvicorn sqlalchemy alembic pydantic-settings
```

## Monorepo (Turborepo)

```
my-monorepo/
├── apps/
│   ├── web/                     # Next.js frontend
│   └── api/                     # Backend API
├── packages/
│   ├── ui/                      # Shared component library
│   ├── db/                      # Shared database package
│   ├── config-ts/               # Shared TS config
│   └── config-eslint/           # Shared ESLint config
├── turbo.json
├── pnpm-workspace.yaml
├── package.json
└── .env
```

**Setup:**
```bash
npx create-turbo@latest my-monorepo
cd my-monorepo
```

## Chrome Extension (Manifest V3)

```
my-extension/
├── src/
│   ├── background/
│   │   └── service-worker.ts
│   ├── content/
│   │   └── content-script.ts
│   ├── popup/
│   │   ├── popup.html
│   │   ├── popup.tsx
│   │   └── popup.css
│   ├── options/
│   │   ├── options.html
│   │   └── options.tsx
│   └── lib/
│       ├── storage.ts
│       └── messaging.ts
├── public/
│   ├── icons/
│   └── manifest.json
├── vite.config.ts
├── tsconfig.json
└── package.json
```

## CLI Tool

```
my-cli/
├── src/
│   ├── index.ts                 # Entry point
│   ├── commands/
│   │   ├── init.ts
│   │   └── run.ts
│   ├── lib/
│   │   ├── config.ts
│   │   └── utils.ts
│   └── types.ts
├── bin/
│   └── cli.js                   # Shebang entry
├── tsconfig.json
├── package.json
└── README.md
```

**Setup:**
```bash
mkdir my-cli && cd my-cli
npm init -y
npm install commander chalk ora
npm install -D typescript @types/node tsx
```
