# GitHub Actions & CI/CD Templates

## Full CI Pipeline (Next.js)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run lint
      - run: npx tsc --noEmit

  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: coverage/

  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run build
        env:
          NEXT_PUBLIC_APP_URL: https://myapp.com
```

## Preview Deployments

```yaml
# .github/workflows/preview.yml
name: Preview Deploy

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run build
      - name: Deploy to Vercel Preview
        id: deploy
        run: |
          npm i -g vercel
          DEPLOY_URL=$(vercel deploy --token=${{ secrets.VERCEL_TOKEN }} --yes)
          echo "url=$DEPLOY_URL" >> $GITHUB_OUTPUT
      - name: Comment PR with preview URL
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          message: |
            🚀 **Preview deployed!**
            ${{ steps.deploy.outputs.url }}
```

## Production Deploy on Merge

```yaml
# .github/workflows/deploy.yml
name: Deploy Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run build
      - name: Deploy to Vercel Production
        run: |
          npm i -g vercel
          vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }} --yes
```

## Cloudflare Workers Deploy

```yaml
# .github/workflows/deploy-cf.yml
name: Deploy Worker

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run build
      - name: Deploy to Cloudflare
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
```

## Security Scanning in CI

```yaml
# .github/workflows/security.yml
name: Security

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: "0 6 * * 1" # Weekly Monday 6am

jobs:
  audit:
    name: Dependency Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm audit --audit-level=high

  secrets-scan:
    name: Secret Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified
```

## Release Automation with Changesets

```bash
npm install -D @changesets/cli @changesets/changelog-github
npx changeset init
```

```json
// .changeset/config.json
{
  "$schema": "https://unpkg.com/@changesets/config@3.0.0/schema.json",
  "changelog": ["@changesets/changelog-github", { "repo": "user/repo" }],
  "commit": false,
  "access": "public",
  "baseBranch": "main"
}
```

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - uses: changesets/action@v1
        with:
          publish: npm run release
          version: npm run version
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Auto-Changelog from Conventional Commits

```yaml
- name: Generate Changelog
  uses: orhun/git-cliff-action@v3
  with:
    config: cliff.toml
    args: --verbose
  env:
    OUTPUT: CHANGELOG.md
```

## Claude Code Review in CI

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Claude Code Review
        run: |
          claude --print "Review the changes in this PR for security, performance, and correctness issues. Be concise." \
            --allowedTools Read,Grep,Glob \
            > review.md
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      - name: Post Review Comment
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          path: review.md
```

## Monorepo CI (Turborepo)

```yaml
# .github/workflows/ci-monorepo.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2 # For turbo change detection
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci

      # Only build/test affected packages
      - run: npx turbo run lint test build --filter="...[HEAD~1]"
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          TURBO_TEAM: ${{ vars.TURBO_TEAM }}
```

## Database Migrations in CI

```yaml
  migrate:
    name: Run Migrations
    runs-on: ubuntu-latest
    environment: production
    needs: [build]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - name: Run Drizzle migrations
        run: npx drizzle-kit migrate
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## Workflow Templates by Project Type

**SaaS App (Next.js + Vercel)**
```
Push to PR → Lint + Test + Type Check → Preview Deploy → Comment URL
Merge to main → Full CI → Migrate DB → Deploy Production
Weekly → Security Audit + Dependency Update
```

**API Service (Hono + Cloudflare Workers)**
```
Push to PR → Lint + Test → Preview Worker
Merge to main → Full CI → Deploy Worker
```

**Library/Package**
```
Push to PR → Lint + Test + Build → Size Check
Merge to main → Changesets Release PR → Publish to npm
```

**Monorepo**
```
Push to PR → Turbo filter affected → Lint + Test + Build (parallel)
Merge to main → Turbo full → Deploy affected services
```

## Optimization Tips

Cache node modules:
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: "npm"
```

Cache Playwright browsers:
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ hashFiles('package-lock.json') }}
```

Cancel redundant runs:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Only run heavy jobs when needed:
```yaml
e2e:
  if: github.event_name == 'push' || contains(github.event.pull_request.labels.*.name, 'run-e2e')
```

## CI Checklist for New Projects

- [ ] Lint check runs on every PR
- [ ] Type check (`tsc --noEmit`) runs on every PR
- [ ] Unit tests run on every PR
- [ ] Build succeeds on every PR
- [ ] Preview deployment on PR (with comment URL)
- [ ] Production deploy on merge to main
- [ ] Dependency audit (weekly schedule)
- [ ] Secret scanning enabled
- [ ] Concurrency limits set (cancel redundant runs)
- [ ] Branch protection: require CI pass before merge
