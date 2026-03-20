# Schedule Patterns — Reference

Full templates, examples, and platform-specific guidance for recurring tasks.

## GitHub Actions — Full Schedule Workflow

```yaml
# .github/workflows/nightly-checks.yml
name: Nightly Checks

on:
  schedule:
    - cron: "0 2 * * *"   # 2am UTC daily
  workflow_dispatch:        # manual trigger always

concurrency:
  group: nightly-${{ github.ref }}
  cancel-in-progress: true

jobs:
  tests:
    name: Full Test Suite
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm test -- --run
      - name: Notify on failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "Nightly tests failed: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  security-audit:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: "npm" }
      - run: npm ci
      - run: npm audit --audit-level=high
```

## GitHub Actions — Weekly Dependency Update

```yaml
# .github/workflows/deps-update.yml
name: Weekly Dependency Update

on:
  schedule:
    - cron: "0 9 * * 1"   # Monday 9am UTC
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: "npm" }
      - run: npm install -g npm-check-updates
      - run: ncu -u
      - run: npm install
      - run: npm test -- --run
      - uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "chore: weekly dependency update"
          title: "chore: weekly dependency update"
          body: "Automated weekly dependency update. Review and merge if tests pass."
          branch: "deps/weekly-update"
          delete-branch: true
          labels: "dependencies"
```

## GitHub Actions — Monthly Security + Code Quality

```yaml
# .github/workflows/monthly-audit.yml
name: Monthly Code Audit

on:
  schedule:
    - cron: "0 0 1 * *"   # first of month at midnight
  workflow_dispatch:

jobs:
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: "npm" }
      - run: npm ci
      - run: npm run lint 2>&1 | tee lint-report.txt
      - run: npx tsc --noEmit 2>&1 | tee type-report.txt
      - run: npm test -- --run --reporter=verbose 2>&1 | tee test-report.txt
      - uses: actions/upload-artifact@v4
        with:
          name: monthly-audit-${{ github.run_number }}
          path: |
            lint-report.txt
            type-report.txt
            test-report.txt
```

## Vercel Cron Jobs

```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 3 * * *"
    },
    {
      "path": "/api/cron/send-digest",
      "schedule": "0 9 * * 1"
    }
  ]
}
```

Route handler must validate the cron secret:

```typescript
// app/api/cron/cleanup/route.ts
import { NextRequest } from "next/server";

export async function GET(req: NextRequest) {
  const authHeader = req.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response("Unauthorized", { status: 401 });
  }

  // Run your scheduled task
  await cleanupExpiredSessions();

  return Response.json({ ok: true, timestamp: new Date().toISOString() });
}
```

Set `CRON_SECRET` in Vercel environment variables. Vercel only invokes crons in production.

## Cloudflare Workers Cron

```typescript
// worker.ts
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    ctx.waitUntil(runScheduledTask(env));
  },
  async fetch(request: Request, env: Env): Promise<Response> {
    return new Response("Worker running");
  },
};

async function runScheduledTask(env: Env) {
  const start = Date.now();
  console.log(JSON.stringify({ event: "cron_start", ts: new Date().toISOString() }));
  // ... task logic ...
  console.log(JSON.stringify({
    event: "cron_complete",
    ts: new Date().toISOString(),
    duration_ms: Date.now() - start
  }));
}
```

```toml
# wrangler.toml
[[triggers]]
crons = ["0 * * * *", "30 12 * * *"]
```

Test locally: `wrangler dev --test-scheduled`

## Failure Notification Patterns

### Slack Webhook (GitHub Actions)
```yaml
- name: Notify failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": ":x: Scheduled job *${{ github.workflow }}* failed",
        "attachments": [{
          "color": "danger",
          "fields": [
            {"title": "Repository", "value": "${{ github.repository }}", "short": true},
            {"title": "Run", "value": "<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View Run>", "short": true}
          ]
        }]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

### Email via Resend (Vercel Cron)
```typescript
import { Resend } from "resend";

export async function notifyFailure(error: Error, task: string) {
  const resend = new Resend(process.env.RESEND_API_KEY);
  await resend.emails.send({
    from: "cron@yourdomain.com",
    to: "alerts@yourdomain.com",
    subject: `[CRON FAILURE] ${task}`,
    text: `Scheduled task "${task}" failed at ${new Date().toISOString()}\n\nError: ${error.message}\n\n${error.stack}`,
  });
}
```

## Scheduled Task Checklist

Before deploying a scheduled task:

- [ ] Task is idempotent (safe to run twice)
- [ ] Explicit timeout defined (not unlimited)
- [ ] Failure alerting connected (Slack/email/Sentry)
- [ ] Manual trigger available (workflow_dispatch or test endpoint)
- [ ] Structured logging in place (timestamps, duration, result)
- [ ] Cron expression validated (use crontab.guru)
- [ ] Timezone considered (GitHub Actions = UTC)
- [ ] Rate limits checked (external APIs called in task)
- [ ] Idling cost checked (Workers/Lambda billing)

## Cron Expression Cheat Sheet

```
Every minute:         * * * * *
Every 5 minutes:      */5 * * * *
Every hour:           0 * * * *
Every day at noon:    0 12 * * *
Every weekday 9am:    0 9 * * 1-5
Every Monday 8am:     0 8 * * 1
First of month noon:  0 12 1 * *
Every quarter:        0 0 1 1,4,7,10 *
```

Validate expressions at: https://crontab.guru
