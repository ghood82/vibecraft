---
name: scheduler
description: "Recurring tasks, cron jobs, scheduled automation, periodic checks, dependency updates on a schedule. Use for scheduling patterns: cron syntax, GitHub Actions schedule, platform cron, task recurrence. Do NOT use for one-time CI runs (use cicd) or monitoring alerts (use monitoring)."
---

# Scheduler — Recurring Task Patterns

Define tasks that run on a schedule: nightly tests, weekly dependency audits, daily code quality sweeps, periodic data syncs. Covers cron syntax, platform-native scheduling, and GitHub Actions `schedule:` triggers.

## When to Activate

- "Run tests every commit, but also nightly"
- "Schedule a weekly dependency update"
- "Add a cron job for periodic cleanup"
- "Make this check run every day at 8am"
- Any recurring, time-based, or interval-based task

## Cron Syntax Quick Reference

```
┌─ minute (0-59)
│ ┌─ hour (0-23)
│ │ ┌─ day of month (1-31)
│ │ │ ┌─ month (1-12)
│ │ │ │ ┌─ day of week (0-6, Sun=0)
│ │ │ │ │
* * * * *

Examples:
  0 9 * * 1-5     → 9am weekdays
  0 2 * * *       → 2am every day
  0 0 * * 0       → midnight Sunday
  */30 * * * *    → every 30 minutes
  0 0 1 * *       → first of every month
```

**Important**: GitHub Actions cron uses UTC. Convert your target timezone.

## Platform-Native Scheduling

### GitHub Actions `schedule:`
```yaml
on:
  schedule:
    - cron: "0 2 * * *"   # nightly at 2am UTC
  workflow_dispatch:        # always add manual trigger
```

### Vercel Cron Jobs (`vercel.json`)
```json
{
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 3 * * *"
    }
  ]
}
```
Validate with `CRON_SECRET` header. Only runs in production.

### Cloudflare Workers (wrangler.toml)
```toml
[[triggers]]
crons = ["0 * * * *"]
```

## Common Recurring Patterns

| Task | Schedule | Notes |
|------|----------|-------|
| Nightly tests | `0 2 * * *` | Off-peak hours |
| Weekly dep update | `0 9 * * 1` | Monday 9am |
| Daily code quality | `0 6 * * *` | Before work hours |
| Hourly health check | `0 * * * *` | Use uptime service instead if available |
| Monthly security audit | `0 0 1 * *` | First of month |
| Pre-release checks | On tag push | Not time-based — event-based |

## Scheduled Task Design

Every scheduled task must have:
1. **Idempotency** — safe to run twice without side effects
2. **Timeout** — explicit max runtime (not unlimited)
3. **Failure alerting** — where does the failure go? Slack/email/Sentry
4. **Manual trigger** — always add `workflow_dispatch` for on-demand runs
5. **Logging** — timestamps + result in output (pair with structured-logging skill)

## Dependency Update Pattern (Recommended)

```yaml
on:
  schedule:
    - cron: "0 9 * * 1"  # Monday 9am
  workflow_dispatch:

jobs:
  update-deps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx npm-check-updates -u
      - run: npm install
      - run: npm test -- --run
      - uses: peter-evans/create-pull-request@v6
        with:
          title: "chore: weekly dependency update"
          branch: "deps/weekly-update"
```

## Reference

Read `references/schedule-patterns.md` for:
- Full GitHub Actions schedule workflow templates
- Vercel/Cloudflare cron implementation examples
- Failure notification patterns (Slack, email, Sentry)
- Scheduled task checklist
- Cron expression generator tips
- Platform cron limits and gotchas
