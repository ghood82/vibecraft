# Log Formats — Reference

Logger setup templates, aggregation integration, sampling, and security-safe patterns.

## Logger Setup Templates

### Node.js / TypeScript — pino (Production Setup)

```typescript
// lib/logger.ts
import pino, { Logger } from "pino";

const isDev = process.env.NODE_ENV === "development";

export const logger: Logger = pino({
  level: process.env.LOG_LEVEL ?? (isDev ? "debug" : "info"),
  base: {
    service: process.env.SERVICE_NAME ?? "app",
    env: process.env.NODE_ENV ?? "development",
    version: process.env.APP_VERSION ?? "unknown",
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  transport: isDev
    ? { target: "pino-pretty", options: { colorize: true } }
    : undefined,
  redact: {
    paths: ["*.password", "*.token", "*.apiKey", "*.authorization", "*.email"],
    censor: "[REDACTED]",
  },
});

// Child logger for request context
export function requestLogger(requestId: string, userId?: string) {
  return logger.child({ requestId, userId });
}
```

Usage in Next.js route handler:
```typescript
// app/api/orders/route.ts
import { logger } from "@/lib/logger";

export async function POST(req: Request) {
  const start = Date.now();
  const reqLog = logger.child({ requestId: crypto.randomUUID() });

  reqLog.info({ method: "POST", path: "/api/orders" }, "Request received");

  try {
    const body = await req.json();
    const order = await createOrder(body);
    reqLog.info({ orderId: order.id, duration_ms: Date.now() - start }, "Order created");
    return Response.json(order);
  } catch (err) {
    reqLog.error({ error: (err as Error).message, duration_ms: Date.now() - start }, "Order creation failed");
    return Response.json({ error: "Order failed" }, { status: 500 });
  }
}
```

### Python — structlog (Production Setup)

```python
# app/core/logging.py
import structlog
import logging
import sys
import os

def setup_logging():
    log_level = os.getenv("LOG_LEVEL", "INFO").upper()
    is_dev = os.getenv("ENV", "production") == "development"

    shared_processors = [
        structlog.contextvars.merge_contextvars,
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
    ]

    if is_dev:
        processors = shared_processors + [
            structlog.dev.ConsoleRenderer(),
        ]
    else:
        processors = shared_processors + [
            structlog.processors.dict_tracebacks,
            structlog.processors.JSONRenderer(),
        ]

    structlog.configure(
        processors=processors,
        wrapper_class=structlog.stdlib.BoundLogger,
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory(),
    )

log = structlog.get_logger()

# Usage
setup_logging()
log.info("order_created", order_id="ord_123", user_id="usr_abc", amount=49.99)
log.error("payment_failed", order_id="ord_123", error=str(e), code="card_declined")

# Request context binding
structlog.contextvars.bind_contextvars(request_id="req_abc", user_id="usr_abc")
log.info("processing_request")
structlog.contextvars.clear_contextvars()
```

### Go — slog (stdlib, Go 1.21+)

```go
package main

import (
    "log/slog"
    "os"
)

func main() {
    env := os.Getenv("ENV")
    var handler slog.Handler

    if env == "development" {
        handler = slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelDebug})
    } else {
        handler = slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelInfo})
    }

    logger := slog.New(handler)
    slog.SetDefault(logger)

    slog.Info("order created", "orderId", orderId, "userId", userId)
    slog.Error("payment failed", "orderId", orderId, "error", err)
}
```

## Session Log Files

For CLI tools, pipelines, or long agent runs:

```typescript
// lib/session-logger.ts
import pino from "pino";
import { mkdirSync } from "fs";

export function createSessionLogger(sessionName = "session") {
  mkdirSync("logs", { recursive: true });
  const ts = new Date().toISOString().replace(/[:.]/g, "-").slice(0, 19);
  const file = `logs/${sessionName}-${ts}.ndjson`;

  const fileLogger = pino(pino.destination(file));
  const consoleLogger = pino({
    transport: { target: "pino-pretty" },
    level: "info",
  });

  // Mirror to both file and console
  return {
    info: (ctx: object, msg: string) => { fileLogger.info(ctx, msg); consoleLogger.info(ctx, msg); },
    warn: (ctx: object, msg: string) => { fileLogger.warn(ctx, msg); consoleLogger.warn(ctx, msg); },
    error: (ctx: object, msg: string) => { fileLogger.error(ctx, msg); consoleLogger.error(ctx, msg); },
    debug: (ctx: object, msg: string) => { fileLogger.debug(ctx, msg); },
    logFile: file,
  };
}
```

Review commands:
```bash
# All errors in today's logs
grep '"level":"error"' logs/*.ndjson | jq .

# Events for specific entity
cat logs/*.ndjson | jq 'select(.orderId == "ord_123")'

# Summary timeline
cat logs/*.ndjson | jq -r '[.ts, .level, .msg] | @tsv' | column -t

# Count by level
cat logs/*.ndjson | jq -r '.level' | sort | uniq -c | sort -rn
```

## Log Aggregation Integration

### Axiom (recommended for Vercel/Cloudflare)
```typescript
import pino from "pino";
import { AxiomTransport } from "@axiomhq/pino";

export const logger = pino({
  level: "info",
  transport: {
    targets: [
      {
        target: "@axiomhq/pino",
        options: {
          dataset: process.env.AXIOM_DATASET,
          token: process.env.AXIOM_TOKEN,
        },
      },
      ...(process.env.NODE_ENV === "development"
        ? [{ target: "pino-pretty" }]
        : []),
    ],
  },
});
```

### Datadog
```bash
# Set env vars
DD_API_KEY=xxx
DD_SITE=datadoghq.com

# Use dd-trace for automatic correlation
npm install dd-trace
```
```typescript
// At app entry point (before imports)
import "dd-trace/init";
```

### BetterStack (Logtail)
```typescript
import { Logtail } from "@logtail/node";
import { LogtailTransport } from "@logtail/pino";
import pino from "pino";

const logtail = new Logtail(process.env.LOGTAIL_SOURCE_TOKEN!);
export const logger = pino({ level: "info" }, new LogtailTransport(logtail));
```

## Sampling for High-Volume Services

Don't log every event at INFO when traffic is high:

```typescript
// Log 1% of successful requests, 100% of errors
function shouldLog(level: string, sampleRate = 0.01): boolean {
  if (level === "error" || level === "warn") return true;
  return Math.random() < sampleRate;
}

if (shouldLog("info")) {
  logger.info({ orderId, duration_ms }, "Request complete");
}
```

## Security-Safe Logging (PII Scrubbing)

```typescript
// Pino redact config
const logger = pino({
  redact: {
    paths: [
      "*.password",
      "*.token",
      "*.secret",
      "*.apiKey",
      "*.authorization",
      "*.email",         // PII
      "*.ssn",           // PII
      "*.creditCard",    // PCI
    ],
    censor: "[REDACTED]",
  },
});

// Manual scrubbing for complex objects
function scrubPII<T extends Record<string, unknown>>(obj: T): T {
  const PII_KEYS = new Set(["email", "password", "token", "ssn", "phone"]);
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [k, PII_KEYS.has(k) ? "[REDACTED]" : v])
  ) as T;
}
```

## Log Review Debugging Checklist

When debugging with logs:

- [ ] Filter to the time window of the incident
- [ ] Start with `level:error` entries — what failed?
- [ ] Trace the request/entity ID across all log entries
- [ ] Look for `warn` entries before the `error` — early signals
- [ ] Check `duration_ms` fields — what was slow?
- [ ] Verify context fields are populated (userId, requestId, etc.)
- [ ] Look for repeated retries (same error recurring)
- [ ] Check for silent swallowing (no error log before failure)
