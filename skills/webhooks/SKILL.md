---
name: webhooks
description: "Webhook and event-driven triggers — respond to GitHub events, API calls, Slack messages, and external signals to auto-start workflows. Use for 'trigger on push', 'webhook handler', 'event-driven', 'notify on deploy'. Do NOT use for manual task execution or polling."
---

# Webhooks — Event-Driven Workflow Triggers

## When to Activate
- User wants workflows to start automatically from external events
- GitHub push/PR/issue events should trigger actions
- Slack messages or API calls should kick off processes
- Need to connect external services to the coding agent

## Event-Driven Architecture

### Event → Handler → Action Pipeline
```
External Event (GitHub push, Slack msg, API call)
  ↓
Webhook Receiver (Express server, Cloudflare Worker, Vercel Function)
  ↓
Event Router (match event type to handler)
  ↓
Action Executor (run tests, deploy, notify, trigger loop-engine)
  ↓
Response + Logging (confirm receipt, log event)
```

### Supported Event Sources

| Source | Events | Integration |
|--------|--------|-------------|
| GitHub | push, pull_request, issues, release, check_suite | Webhooks API |
| Slack | message, reaction, slash_command | Events API / Bolt |
| Vercel | deployment, deployment_ready, deployment_error | Deploy hooks |
| Stripe | payment_succeeded, subscription_updated | Webhooks |
| Custom API | Any POST/PUT to your endpoint | Express/Hono |
| Cron | Scheduled triggers | See scheduling skill |

## Implementation Protocol

1. **Define** — Which events trigger which workflows?
2. **Secure** — Verify webhook signatures (HMAC, tokens)
3. **Route** — Map event types to handler functions
4. **Execute** — Run the workflow (may delegate to other skills)
5. **Log** — Record event, action taken, result (see logging skill)
6. **Respond** — Return 200 quickly, process async

Read `references/webhook-implementations.md` for:
- GitHub webhook server with signature verification
- Slack bot with event subscriptions
- Generic webhook receiver with routing
- Cloudflare Worker webhook handler
- Security best practices (HMAC, replay protection, rate limiting)

## Webhook Security Checklist
- Always verify signatures (never trust raw payloads)
- Use HTTPS only
- Implement replay protection (timestamp + nonce)
- Rate limit incoming events
- Log all events for audit trail
- Timeout long-running handlers (respond 200 first, process async)

## Integration with Other Skills
- **loop-engine**: Webhook events can trigger iterative fix cycles
- **scheduling**: Webhooks complement cron — events vs time-based
- **logging**: All webhook events are logged
- **deployment**: Deploy webhooks trigger verification loops
- **orchestrator**: Events route through orchestrator to right skill