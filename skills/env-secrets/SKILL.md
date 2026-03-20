---
name: env-secrets
description: "Environment variables, secrets management, .env configuration. Use for env/secrets setup. Do NOT use for deployment, CI/CD pipelines, or application config logic."
---

# Environment & Secrets Management

Handle configuration safely across environments. Never leak secrets, always validate, always type.

## .env File Hierarchy (Precedence Order)
```
.env.local          # Local overrides (NEVER committed, highest priority)
.env.development    # Dev-specific defaults
.env.staging        # Staging-specific defaults
.env.production     # Production-specific defaults
.env                # Shared defaults (lowest priority, committed)
```

Always commit a `.env.example` with all required vars but no real values.

## Typed Environment Variables (Zod)

```typescript
// src/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  AUTH_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string().startsWith("sk_"),
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
});

const parsed = envSchema.safeParse(process.env);
if (!parsed.success) {
  console.error("Invalid environment variables:", parsed.error.flatten().fieldErrors);
  throw new Error("Invalid environment variables");
}
export const env = parsed.data;
```

For Next.js, use `@t3-oss/env-nextjs` for server/client separation with build-time safety.

## Security Rules

| Type | Example | Where | Public? |
|------|---------|-------|---------|
| Database URLs | `postgres://...` | Platform secrets | Never |
| Secret API keys | `sk_live_...` | Platform secrets | Never |
| Public API keys | `pk_live_...` | `NEXT_PUBLIC_*` | Yes |
| Webhook secrets | `whsec_...` | Platform secrets | Never |
| Auth secrets | JWT signing key | Platform secrets | Never |

## Platform-Specific Secrets

**Vercel**: `vercel env add KEY production` / `vercel env pull .env.local`
**Cloudflare**: `npx wrangler secret put KEY` (encrypted, not in wrangler.toml)
**Supabase**: `npx supabase secrets set KEY=value`
**GitHub Actions**: Use `${{ secrets.KEY }}` in workflow files

## If a Secret is Leaked

1. **Rotate immediately** — new key on provider dashboard
2. **Update all environments** — platform secrets, CI/CD, local
3. **Check git history** — old secret is in git forever if committed
4. **Audit usage** — check provider dashboard for unauthorized use
5. **BFG Repo-Cleaner** — scrub from git history (last resort)

## Setup Checklist

- [ ] `.env.example` committed with all vars documented
- [ ] `.env.local` in `.gitignore`
- [ ] Zod validation in `src/env.ts`
- [ ] Platform secrets configured (Vercel/Cloudflare/Supabase)
- [ ] Separate values for dev/staging/production
- [ ] No secrets in client-accessible code
- [ ] CI/CD has all required secrets
- [ ] Team knows to copy `.env.example` → `.env.local`

## Reference Docs

Read `references/env-config-examples.md` for full .env.example template, T3 Env setup, Cloudflare Workers config, multi-env config pattern, and .gitignore rules.
