---
name: cicd
description: "CI/CD pipelines, GitHub Actions, automated deploys, releases. Use for pipeline setup. Do NOT use for manual deployment or code review."
---

# CI/CD Pipeline Generation

Automate everything: lint, test, build, deploy. Covers GitHub Actions (primary), plus GitLab CI and headless Claude Code in CI.

## Workflow Fundamentals

CI/CD pipelines run on events (push, PR, schedule) and execute jobs in sequence or parallel:
- **Lint**: Code style, type checking
- **Test**: Unit, integration, E2E tests
- **Build**: Compile, bundle, optimize
- **Deploy**: Preview or production environments
- **Security**: Dependency audit, secret scanning

## Process: Design & Implement Pipeline

1. **Understand**: What environments? What triggers? What checks matter?
2. **Choose tools**: GitHub Actions is standard; consider GitLab CI or Vercel
3. **Design jobs**: Lint → Test → Build → Deploy (with parallelization where safe)
4. **Add caching**: Node modules, Playwright browsers, build artifacts
5. **Set concurrency**: Cancel redundant runs to save credits
6. **Protect branches**: Require CI pass, code review, status checks before merge
7. **Add secrets**: API tokens for deployment, npm registry, analytics
8. **Monitor**: Review failed runs, tune alert thresholds

## CI/CD Patterns by Project Type

### SaaS App (Next.js + Vercel)
```
Push to PR → Lint + Test + Type Check → Preview Deploy → Comment URL
Merge to main → Full CI → Migrate DB → Deploy Production
Weekly → Security Audit + Dependency Update
```

### API Service (Hono + Cloudflare Workers)
```
Push to PR → Lint + Test → Preview Worker
Merge to main → Full CI → Deploy Worker
```

### Library/Package
```
Push to PR → Lint + Test + Build → Size Check
Merge to main → Changesets Release PR → Publish to npm
```

### Monorepo
```
Push to PR → Turbo filter affected → Lint + Test + Build
Merge to main → Turbo full → Deploy affected services
```

## GitHub Actions Syntax Quick Reference

**Triggers**: `on: [push, pull_request]` with branch/path filters
**Concurrency**: Cancel redundant runs with group + ref
**Jobs**: Run in parallel by default, `needs:` for dependencies
**Steps**: Sequential commands with `run:` or use `actions/`
**Caching**: `actions/setup-node@v4` with `cache: "npm"`
**Artifacts**: Upload/download with `actions/upload-artifact`
**Secrets**: Reference with `${{ secrets.NAME }}`
**Outputs**: Pass data between steps with `$GITHUB_OUTPUT`

## Reference Templates

Read `references/github-actions-templates.md` for:
- Full CI pipeline (lint, test, build, E2E)
- Preview deployments with PR comments
- Production deploy on merge
- Cloudflare Workers deploy
- Security scanning (audit, secrets)
- Release automation (Changesets, git-cliff)
- Claude Code review in CI
- Monorepo setup with Turborepo
- Database migrations
- Caching and optimization tips
- CI checklist for new projects

## Common CI Mistakes to Avoid

- **No caching**: Every install from scratch (slow, wastes credits)
- **Redundant runs**: PR + push both trigger (set concurrency)
- **E2E on every PR**: E2E is slow; run on push to main or with label
- **No branch protection**: CI passes but unreviewed code merges anyway
- **Secrets in logs**: Never echo secrets; use environment variables
- **No fail-fast**: Let all jobs run even after one fails
- **Missing env vars**: Build succeeds locally but fails in CI

---

**Next Steps**: Choose your project type above, then read the reference templates to set up a basic CI pipeline.
