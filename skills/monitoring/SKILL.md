---
name: monitoring
description: "Error tracking, logging, health checks, alerts, analytics. Use for monitoring setup. Do NOT use for performance or security scanning."
---

# Monitoring & Observability

Know what's happening in production. Covers error tracking, structured logging, health checks, analytics, and alerting.

## Three Pillars: Logs, Metrics, Traces

- **Logs**: What happened? (Structured logging, aggregation)
- **Metrics**: How much/how fast? (Core Web Vitals, counters)
- **Traces**: Why is it slow? (Distributed tracing, spans)

## Process: Set Up Monitoring

1. Error tracking (Sentry)
2. Structured logging (JSON to aggregator)
3. Health checks (`/api/health` endpoint)
4. Core Web Vitals reporting (LCP, INP, CLS)
5. Custom metrics (business events, analytics)
6. Alerting (on error rate, response time, health)
7. Dashboards (visualize trends)

## Error Tracking (Sentry)

```bash
npx @sentry/wizard@latest -i nextjs
```

Set user context, add breadcrumbs, use Error Boundary, manual capture in try/catch blocks.

## Structured Logging

Output JSON in production, pretty-print in dev:

```typescript
logger.info("Order created", { orderId, items });
logger.error("Payment failed", { orderId, error: msg });
```

Log levels: `debug` → `info` → `warn` → `error`. Always include context in errors.

## Health Checks

Endpoint `/api/health` checks database, Redis, external APIs. Returns 200 if healthy, 503 if degraded.

## Core Web Vitals & Analytics

Client-side reporting of LCP, INP, CLS, FCP, TTFB. Use `navigator.sendBeacon` for reliability.

## Alerting

Alert on: error rate > 5%, p95 latency > 3s, health failures, disk > 85%, zero traffic.
Don't alert on: expected spikes, single errors, dev/staging.

## Monitoring Stack Options

**Minimal**: Sentry, BetterStack uptime, Plausible analytics, platform logs
**Production**: Sentry, Axiom/Datadog logging, Grafana metrics, PostHog analytics

## Checklist: Before Launch

- [ ] Error tracking (Sentry)
- [ ] Health check endpoint
- [ ] Structured logging
- [ ] Uptime monitoring
- [ ] Core Web Vitals
- [ ] Alert notifications

## Reference Implementation Details

Read `references/monitoring-patterns.md` for:
- Sentry setup with Next.js, error boundaries, context
- Logger implementation with child loggers
- Health check code examples
- Core Web Vitals reporting code
- Alert rule templates
- Production readiness checklist

---

**Next Steps**: Set up Sentry + health check before launch. Add structured logging.
