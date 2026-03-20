# Advanced Monitoring Patterns

## Request Tracing Middleware

```typescript
// middleware/tracing.ts
import { logger } from "@/lib/logger";

export function tracingMiddleware(handler: Function) {
  return async (req: Request) => {
    const requestId = req.headers.get("x-request-id") || crypto.randomUUID();
    const startTime = performance.now();
    const url = new URL(req.url);

    const reqLogger = logger.child({
      requestId,
      method: req.method,
      path: url.pathname,
    });

    reqLogger.info("Request started");

    try {
      const response = await handler(req, { requestId, logger: reqLogger });
      const duration = performance.now() - startTime;

      reqLogger.info("Request completed", {
        status: response.status,
        durationMs: Math.round(duration),
      });

      // Add tracing headers to response
      response.headers.set("x-request-id", requestId);
      response.headers.set("x-response-time", `${Math.round(duration)}ms`);

      return response;
    } catch (error) {
      const duration = performance.now() - startTime;
      reqLogger.error("Request failed", {
        error: error instanceof Error ? error.message : "Unknown error",
        durationMs: Math.round(duration),
      });
      throw error;
    }
  };
}
```

## Custom Metrics Collection

```typescript
// lib/metrics.ts
class MetricsCollector {
  private counters = new Map<string, number>();
  private gauges = new Map<string, number>();
  private histograms = new Map<string, number[]>();

  increment(name: string, value = 1, tags?: Record<string, string>) {
    const key = this.buildKey(name, tags);
    this.counters.set(key, (this.counters.get(key) || 0) + value);
  }

  gauge(name: string, value: number, tags?: Record<string, string>) {
    const key = this.buildKey(name, tags);
    this.gauges.set(key, value);
  }

  histogram(name: string, value: number, tags?: Record<string, string>) {
    const key = this.buildKey(name, tags);
    const existing = this.histograms.get(key) || [];
    existing.push(value);
    this.histograms.set(key, existing);
  }

  // Calculate percentiles for histograms
  percentile(name: string, p: number, tags?: Record<string, string>): number | null {
    const key = this.buildKey(name, tags);
    const values = this.histograms.get(key);
    if (!values || values.length === 0) return null;

    const sorted = [...values].sort((a, b) => a - b);
    const index = Math.ceil((p / 100) * sorted.length) - 1;
    return sorted[index];
  }

  private buildKey(name: string, tags?: Record<string, string>): string {
    if (!tags) return name;
    const tagStr = Object.entries(tags).map(([k, v]) => `${k}:${v}`).sort().join(",");
    return `${name}{${tagStr}}`;
  }

  // Flush metrics to your backend
  async flush() {
    const payload = {
      counters: Object.fromEntries(this.counters),
      gauges: Object.fromEntries(this.gauges),
      histograms: Object.fromEntries(
        [...this.histograms].map(([k, v]) => [k, {
          count: v.length,
          min: Math.min(...v),
          max: Math.max(...v),
          avg: v.reduce((a, b) => a + b, 0) / v.length,
          p50: this.percentile(k.split("{")[0], 50),
          p95: this.percentile(k.split("{")[0], 95),
          p99: this.percentile(k.split("{")[0], 99),
        }])
      ),
      timestamp: Date.now(),
    };

    await fetch("/api/metrics", {
      method: "POST",
      body: JSON.stringify(payload),
      keepalive: true,
    });

    // Reset after flush
    this.counters.clear();
    this.histograms.clear();
  }
}

export const metrics = new MetricsCollector();

// Usage
metrics.increment("api.requests", 1, { method: "GET", path: "/users" });
metrics.histogram("api.latency", 142, { path: "/users" });
metrics.gauge("active.connections", 23);
```

## Uptime Monitoring

### Simple Cron Check (Cloudflare Worker)
```typescript
// Scheduled worker that pings your endpoints
export default {
  async scheduled(event: ScheduledEvent, env: Env) {
    const endpoints = [
      { url: "https://myapp.com/api/health", name: "API" },
      { url: "https://myapp.com", name: "Frontend" },
    ];

    for (const endpoint of endpoints) {
      const start = Date.now();
      try {
        const res = await fetch(endpoint.url, { method: "GET" });
        const latency = Date.now() - start;

        if (!res.ok) {
          await alertDown(env, endpoint.name, `HTTP ${res.status}`, latency);
        } else {
          await recordUp(env, endpoint.name, latency);
        }
      } catch (error) {
        await alertDown(env, endpoint.name, error.message, Date.now() - start);
      }
    }
  },
};

async function alertDown(env: Env, name: string, reason: string, latencyMs: number) {
  // Send to Slack, email, PagerDuty, etc.
  await fetch(env.SLACK_WEBHOOK_URL, {
    method: "POST",
    body: JSON.stringify({
      text: `🔴 ${name} is DOWN: ${reason} (${latencyMs}ms)`,
    }),
  });
}
```

## Sentry Advanced Configuration

```typescript
// Custom error grouping
Sentry.init({
  dsn: "...",
  beforeSend(event) {
    // Don't send expected errors
    if (event.exception?.values?.[0]?.type === "AbortError") {
      return null;
    }

    // Scrub PII
    if (event.request?.headers) {
      delete event.request.headers["Authorization"];
      delete event.request.headers["Cookie"];
    }

    return event;
  },

  // Custom fingerprinting for better grouping
  beforeSendTransaction(event) {
    // Group all /users/:id routes together
    if (event.transaction?.match(/^\/users\/\w+$/)) {
      event.transaction = "/users/:id";
    }
    return event;
  },

  // Performance monitoring
  tracesSampler({ name, parentSampled }) {
    // Always trace health checks at a low rate
    if (name.includes("/health")) return 0.01;
    // Higher rate for payment flows
    if (name.includes("/payment")) return 1.0;
    // Default
    return 0.1;
  },
});
```

## Status Page Pattern

```typescript
// app/api/status/route.ts
export async function GET() {
  const services = await checkAllServices();

  const overallStatus = services.every(s => s.status === "operational")
    ? "operational"
    : services.some(s => s.status === "major_outage")
    ? "major_outage"
    : "degraded";

  return Response.json({
    status: overallStatus,
    updated_at: new Date().toISOString(),
    services: services.map(s => ({
      name: s.name,
      status: s.status,
      latency_ms: s.latencyMs,
      last_incident: s.lastIncident,
    })),
  });
}

async function checkAllServices() {
  return Promise.all([
    checkService("API", "/api/health"),
    checkService("Database", async () => { await db.execute(sql`SELECT 1`); }),
    checkService("Auth", "/api/auth/health"),
    checkService("Storage", async () => { await r2.head("health-check"); }),
  ]);
}
```

## Log Aggregation with Axiom

```typescript
// lib/axiom.ts
import { Axiom } from "@axiomhq/js";

const axiom = new Axiom({ token: process.env.AXIOM_TOKEN! });

export function logToAxiom(events: Record<string, unknown>[]) {
  axiom.ingest("my-app", events);
}

// Middleware that sends structured logs to Axiom
export function axiomMiddleware(handler: Function) {
  return async (req: Request) => {
    const start = Date.now();
    const response = await handler(req);

    logToAxiom([{
      _time: new Date().toISOString(),
      method: req.method,
      url: req.url,
      status: response.status,
      duration_ms: Date.now() - start,
      user_agent: req.headers.get("user-agent"),
      cf_ray: req.headers.get("cf-ray"),
    }]);

    return response;
  };
}
```

## Dashboard Queries (Example for Axiom/Grafana)

```
// Error rate over time
| where status >= 500
| summarize count() by bin(_time, 5m)

// Slowest endpoints
| summarize avg(duration_ms), p95=percentile(duration_ms, 95) by url
| order by p95 desc
| take 10

// Error breakdown
| where status >= 400
| summarize count() by status, url
| order by count_ desc

// Geographic distribution
| summarize count() by cf_colo
| order by count_ desc
```

## Feature Flags with Monitoring

```typescript
// Track feature flag usage alongside metrics
function useFeatureFlag(flag: string): boolean {
  const enabled = getFeatureFlag(flag);

  // Track flag evaluation
  metrics.increment("feature_flag.evaluated", 1, {
    flag,
    result: String(enabled),
  });

  return enabled;
}

// Monitor feature flag impact
if (useFeatureFlag("new_checkout_flow")) {
  metrics.increment("checkout.started", 1, { variant: "new" });
  // new flow
} else {
  metrics.increment("checkout.started", 1, { variant: "control" });
  // old flow
}
```
