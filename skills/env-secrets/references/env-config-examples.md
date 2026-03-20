# Environment Configuration Examples

## .env.example Template
```bash
# .env.example — Copy to .env.local and fill in values

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/myapp

# Auth
AUTH_SECRET=generate-with-openssl-rand-base64-32
NEXTAUTH_URL=http://localhost:3000

# External APIs
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
OPENAI_API_KEY=sk-...

# Public (safe to expose to browser)
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
NEXT_PUBLIC_POSTHOG_KEY=phc_...
```

## .gitignore Rules
```gitignore
# Environment files with real secrets
.env.local
.env.*.local
.env.development.local
.env.production.local

# NEVER ignore .env.example — it's the template
!.env.example
```

## Zod Validation (Full)
```typescript
// src/env.ts
import { z } from "zod";

const envSchema = z.object({
  // Server-only
  DATABASE_URL: z.string().url(),
  AUTH_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string().startsWith("sk_"),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith("whsec_"),
  OPENAI_API_KEY: z.string().startsWith("sk-"),

  // Public (client-safe)
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
  NEXT_PUBLIC_SUPABASE_ANON_KEY: z.string(),
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().startsWith("pk_"),

  // Optional with defaults
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  PORT: z.coerce.number().default(3000),
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
});

const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error("❌ Invalid environment variables:");
  console.error(parsed.error.flatten().fieldErrors);
  throw new Error("Invalid environment variables");
}

export const env = parsed.data;
```

## T3 Env (Next.js)
```typescript
// src/env.ts
import { createEnv } from "@t3-oss/env-nextjs";
import { z } from "zod";

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    AUTH_SECRET: z.string().min(32),
    STRIPE_SECRET_KEY: z.string(),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
    NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
  },
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    AUTH_SECRET: process.env.AUTH_SECRET,
    STRIPE_SECRET_KEY: process.env.STRIPE_SECRET_KEY,
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
    NEXT_PUBLIC_SUPABASE_URL: process.env.NEXT_PUBLIC_SUPABASE_URL,
  },
});
```

## Cloudflare Workers Config
```toml
# wrangler.toml — Non-secret config only
[vars]
ENVIRONMENT = "production"
APP_NAME = "my-app"

# Per-environment overrides
[env.staging.vars]
ENVIRONMENT = "staging"
```

```bash
# Secrets (encrypted, not in wrangler.toml)
npx wrangler secret put STRIPE_SECRET_KEY
npx wrangler secret put DATABASE_URL
npx wrangler secret list
```

Access in Worker:
```typescript
export default {
  async fetch(request: Request, env: Env) {
    // env.STRIPE_SECRET_KEY — from wrangler secret
    // env.ENVIRONMENT — from wrangler.toml [vars]
  },
};
```

## Vercel CLI
```bash
vercel env add DATABASE_URL production
vercel env add DATABASE_URL preview
vercel env add DATABASE_URL development
vercel env pull .env.local  # Download all env vars
vercel env ls
```

## Supabase CLI
```bash
npx supabase secrets set STRIPE_SECRET_KEY=sk_live_xxx
npx supabase secrets list
npx supabase secrets unset STRIPE_SECRET_KEY
```

## GitHub Actions
```yaml
jobs:
  deploy:
    environment: production
    steps:
      - run: echo "Deploying..."
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          STRIPE_KEY: ${{ secrets.STRIPE_SECRET_KEY }}
```

## Multi-Environment Config Pattern
```typescript
// config/index.ts
import { env } from "@/env";

export const config = {
  app: {
    name: "My App",
    url: env.NEXT_PUBLIC_APP_URL,
    isDev: env.NODE_ENV === "development",
    isProd: env.NODE_ENV === "production",
  },
  db: {
    url: env.DATABASE_URL,
    poolSize: env.NODE_ENV === "production" ? 20 : 5,
  },
  stripe: {
    secretKey: env.STRIPE_SECRET_KEY,
    publishableKey: env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY,
    webhookSecret: env.STRIPE_WEBHOOK_SECRET,
  },
  email: {
    from: "noreply@myapp.com",
    replyTo: "support@myapp.com",
  },
} as const;
```

## Anti-Patterns
```typescript
// ❌ Hardcoded secrets
const STRIPE_KEY = "sk_live_abc123";

// ❌ Secret in client code
const secret = process.env.STRIPE_SECRET_KEY; // In a client component

// ❌ Secret in git
// Committed .env.local with real values

// ❌ Logging secrets
console.log("Config:", process.env); // Dumps everything

// ❌ Secrets in error messages
throw new Error(`Auth failed with key: ${apiKey}`);
```

## Secret Leak Check
```bash
git log --all -p | grep -iE "(sk_live|sk_test|password|secret|api_key)" | head -20
```
