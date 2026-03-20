---
name: deployment
description: "Deploy to Vercel, Cloudflare, Docker, any platform. Use when asked to deploy, ship, launch, push to prod, preview deploy, set up CI/CD, configure domain. Do NOT use for CI/CD pipeline generation (use cicd) or container patterns (read references/docker.md)."
---

# Deployment

Ship code to production on any platform. Safety first, speed second.

## Pre-Deployment Checklist

Before deploying ANYTHING, verify:

1. **Build passes** — `npm run build` (or equivalent) exits cleanly
2. **Tests pass** — All tests green, no skipped critical tests
3. **Env vars set** — All required environment variables configured in target
4. **Secrets secure** — No hardcoded secrets, `.env` not committed
5. **Security scan** — No critical vulnerabilities (route to security skill if needed)
6. **Database ready** — Migrations applied, backups taken for production
7. **Branch correct** — Deploying from the right branch (main/production)

## Platform Detection

Auto-detect the target platform from project files:

| File / Config | Platform | Deploy Command |
|--------------|----------|---------------|
| `vercel.json` or Vercel project | Vercel | `vercel --prod` |
| `wrangler.toml` | Cloudflare Workers | `wrangler deploy` |
| `Dockerfile` | Docker / any container host | `docker build && docker push` |
| `fly.toml` | Fly.io | `fly deploy` |
| `railway.json` or Railway project | Railway | `railway up` |
| `render.yaml` | Render | Git push (auto-deploy) |
| `netlify.toml` | Netlify | `netlify deploy --prod` |

If no config detected, ask the user once: "Where do you want to deploy?"

## Deployment Strategies

### Standard Deploy (Default)
For most cases: push to production, let the platform handle it.
```bash
# Vercel
vercel --prod

# Cloudflare
wrangler deploy

# Docker
docker build -t app:latest . && docker push registry/app:latest
```

### Zero-Downtime Deploy
For production apps with users:
- **Blue-Green**: Deploy new version alongside old, switch traffic atomically
- **Canary**: Route 5% of traffic to new version, monitor, then roll out
- **Rolling**: Replace instances one at a time

### Rollback
When things go wrong:
```bash
# Vercel — instant rollback to previous deployment
vercel rollback

# Cloudflare — rollback to previous version
wrangler rollback

# Docker — redeploy previous tag
docker pull registry/app:previous && docker tag registry/app:previous registry/app:latest
```

## Environment Management

Three environments minimum:

| Environment | Purpose | Deployed From | URL Pattern |
|------------|---------|--------------|-------------|
| Development | Local dev | localhost | `localhost:3000` |
| Preview/Staging | Test before prod | Feature branches | `preview-*.vercel.app` |
| Production | Live users | `main` branch | `app.example.com` |

### Environment Variables
- **Never** share production secrets with development
- Use platform-specific env var management (Vercel dashboard, wrangler secret)
- Document all required env vars in `.env.example`

## Domain & DNS

### Custom Domain Setup
1. Add domain in hosting platform dashboard
2. Configure DNS: CNAME for subdomains, A record for apex
3. Wait for DNS propagation (up to 48 hours, usually minutes)
4. Verify SSL certificate is active
5. Set up redirects (www → apex or vice versa)

## CI/CD Pipeline

### GitHub Actions (Recommended)
```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
      - run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

## Post-Deployment

After every production deploy:
1. **Smoke test** — Hit the main endpoints, verify they respond
2. **Monitor** — Check error rates for 15 minutes
3. **Notify** — Post to team channel: "Deployed v1.2.3 — [changelog link]"
4. **Document** — Update changelog if not auto-generated

Read `references/vercel.md` for Vercel-specific deployment details.
Read `references/cloudflare.md` for Cloudflare-specific deployment details.
