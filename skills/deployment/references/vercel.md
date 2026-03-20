# Vercel Deployment Reference

## Project Configuration

### vercel.json
```json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "regions": ["iad1"],
  "env": {
    "DATABASE_URL": "@database-url"
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "no-store" }
      ]
    }
  ],
  "redirects": [
    { "source": "/old-page", "destination": "/new-page", "permanent": true }
  ],
  "rewrites": [
    { "source": "/api/:path*", "destination": "https://api.example.com/:path*" }
  ]
}
```

## Next.js App Router Deployment

### Server Components
- Run at build time (static) or request time (dynamic) on Vercel's infrastructure
- Use `export const dynamic = "force-dynamic"` for pages that need fresh data
- Use `export const revalidate = 3600` for ISR (regenerate every hour)

### Server Actions
- Deployed as serverless functions automatically
- 60-second timeout on Hobby, 300 seconds on Pro
- Use `"use server"` directive at top of file or function

### Edge Runtime
```typescript
// Use edge for low-latency, globally distributed endpoints
export const runtime = "edge";

export async function GET(request: Request) {
  return new Response("Hello from the edge!");
}
```

### Middleware
- Runs on Edge by default (< 1MB limit)
- Use for auth checks, redirects, geolocation, A/B testing
- Cannot access Node.js APIs (no `fs`, `path`, etc.)

## Environment Variables

### Setting via CLI
```bash
# Add a secret (encrypted)
vercel env add DATABASE_URL production
vercel env add DATABASE_URL preview
vercel env add DATABASE_URL development

# Pull env vars for local development
vercel env pull .env.local
```

### Setting via Dashboard
Vercel Dashboard → Project → Settings → Environment Variables

### Scoping
- **Production**: Only available in production deployments
- **Preview**: Available in all preview deployments (branches, PRs)
- **Development**: Available when using `vercel dev`

## Custom Domains

```bash
# Add a custom domain
vercel domains add example.com

# Verify DNS is configured correctly
vercel domains inspect example.com
```

DNS Configuration:
- **Apex domain**: A record → `76.76.21.21`
- **Subdomain**: CNAME → `cname.vercel-dns.com`
- **Wildcard**: CNAME `*.example.com` → `cname.vercel-dns.com`

## Preview Deployments

Every push to a non-production branch creates a preview deployment:
- Unique URL: `project-git-branch-team.vercel.app`
- Comment on PR with preview link (GitHub integration)
- Same environment as production (with preview env vars)

## Vercel CLI Commands

```bash
# Deploy to preview
vercel

# Deploy to production
vercel --prod

# View deployment logs
vercel logs <url>

# List recent deployments
vercel ls

# Rollback to previous deployment
vercel rollback

# Pull project settings
vercel pull

# Link local directory to Vercel project
vercel link
```

## Build Optimization

- **Image Optimization**: Use `next/image` — Vercel handles resizing, WebP conversion
- **ISR**: Incremental Static Regeneration for pages that change occasionally
- **Edge Config**: Use `@vercel/edge-config` for feature flags and runtime config
- **Caching**: Vercel caches static assets at the edge automatically
- **Analytics**: Enable Web Vitals monitoring in project settings

## Monorepo Deployment

```json
// vercel.json at repo root
{
  "projects": [
    {
      "src": "apps/web",
      "use": "@vercel/next"
    }
  ]
}
```

Or set Root Directory in Vercel Dashboard → Project → Settings → General.

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Build fails with module not found | Missing dependency | Check package.json, run `npm ci` |
| Environment variable undefined | Not set for environment | Add in Vercel dashboard, redeploy |
| 504 Gateway Timeout | Function exceeds time limit | Optimize function, use streaming, upgrade plan |
| Edge function too large | > 1MB bundle | Reduce dependencies, use Node runtime instead |
| ISR not updating | Stale cache | Use `revalidateTag()` or `revalidatePath()` |
