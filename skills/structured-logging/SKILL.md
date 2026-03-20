---
name: structured-logging
description: "Write structured logs with timestamps, log levels, session log files, and review logs for debugging. Use for: adding logging to code, log formatting, session logs, debug log review, log levels (debug/info/warn/error), structured JSON output. Do NOT use for monitoring dashboards (use monitoring) or error tracking services like Sentry (use monitoring)."
---

# Structured Logging

Real logging with timestamps, levels, and structured output — not console.log("here") or PROGRESS.md checkpoints. Covers writing logs, reviewing logs, and session log files.

## When to Activate

- "Add logging to this service"
- "I need to review the logs from last session"
- "Log this event with timestamp and context"
- "Set up structured logging for this API"
- "What log level should I use here?"

## Log Levels

```
DEBUG   → dev-only detail. Never in production unless debugging a specific issue.
INFO    → normal operation events worth knowing. System is working as expected.
WARN    → unexpected situation but recoverable. Something to watch.
ERROR   → operation failed. Needs attention. Always include error + context.
```

Rule: If you're adding a `console.log`, decide which level it actually is, then use the right call.

## Structured Log Format

Every log entry: `timestamp + level + message + context object`

```json
{"ts":"2025-03-19T14:23:11.042Z","level":"info","msg":"Order created","orderId":"ord_9xk2","userId":"usr_abc","amount":49.99}
{"ts":"2025-03-19T14:23:11.890Z","level":"error","msg":"Payment failed","orderId":"ord_9xk2","error":"Card declined","code":"card_declined"}
```

Never: `console.log("order created", orderId)`
Always: structured object with consistent keys.

## Implementation Patterns

### Node.js / TypeScript (pino — recommended)
```bash
npm install pino pino-pretty
```
```typescript
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  transport: process.env.NODE_ENV === "development"
    ? { target: "pino-pretty" }
    : undefined,
});

// Usage
logger.info({ orderId, userId }, "Order created");
logger.error({ orderId, error: err.message }, "Payment failed");

// Child loggers for request context
const reqLogger = logger.child({ requestId, userId });
reqLogger.info("Processing request");
```

### Python (structlog — recommended)
```bash
pip install structlog
```
```python
import structlog

log = structlog.get_logger()

log.info("order_created", order_id=order_id, user_id=user_id, amount=amount)
log.error("payment_failed", order_id=order_id, error=str(e), code=error_code)
```

### Session Log Files

For CLI tools, long-running processes, or agent pipelines — write to a file:

```typescript
const sessionId = new Date().toISOString().replace(/[:.]/g, "-");
const sessionLog = pino(pino.destination(`logs/session-${sessionId}.ndjson`));
```

Review session logs: `cat logs/session-*.ndjson | jq .` or `grep '"level":"error"' logs/*.ndjson`

## What to Log

| Log | Level | Required Fields |
|-----|-------|----------------|
| Request received | `info` | method, path, userId |
| Operation complete | `info` | operation, duration_ms, result |
| Retry attempt | `warn` | attempt, maxAttempts, reason |
| Validation failure | `warn` | field, value, rule |
| Unhandled error | `error` | error message, stack, context |
| Startup/shutdown | `info` | service, version, env |

## What NOT to Log

- API keys, tokens, passwords, PII (email, SSN, CC numbers)
- Full request bodies (may contain secrets)
- Full stack traces for expected errors (404, validation)
- Excessive debug logs in production (use sampling or feature flags)

## Reviewing Logs for Debugging

```bash
# All errors in session
grep '"level":"error"' logs/session-*.ndjson | jq .

# Events for a specific order
cat logs/*.ndjson | jq 'select(.orderId == "ord_9xk2")'

# Timeline for a request
cat logs/*.ndjson | jq 'select(.requestId == "req_abc") | {ts, level, msg}'

# Count by level
cat logs/*.ndjson | jq -r '.level' | sort | uniq -c
```

## Reference

Read `references/log-formats.md` for:
- Logger setup templates (Node, Python, Go)
- Log aggregation integration (Axiom, Datadog, BetterStack)
- Sampling strategies for high-volume services
- Security-safe logging patterns (scrubbing PII)
- Log review checklists for debugging sessions
