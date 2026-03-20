---
name: daemon-mode
description: "Persistent background agent — run 24/7 monitoring, auto-respond to events, maintain long-running sessions across reboots. Use for 'keep running', 'background agent', 'always-on', 'daemon', 'persistent process'. Do NOT use for one-shot tasks or interactive sessions."
---

# Daemon Mode — Persistent Background Execution

## When to Activate
- User wants the agent running continuously in the background
- Need to monitor repos, inboxes, or services 24/7
- Auto-respond to events without manual session starts
- Maintain state across system reboots

## Daemon Architecture

### Process Lifecycle
```
INIT → RUNNING → [SLEEPING | PROCESSING] → SHUTDOWN
  ↑                                           |
  └───────── AUTO-RESTART (on crash) ─────────┘
```

### Core Components
1. **Process Manager** — Keeps the agent alive (PM2, systemd, launchd)
2. **Event Loop** — Polls or listens for triggers
3. **State Store** — Persists state across restarts
4. **Health Monitor** — Self-checks, auto-restart on failure
5. **Log Rotator** — Manages output without filling disk

## Implementation Options

| Method | Platform | Best For |
|--------|----------|----------|
| PM2 | Node.js / Any | Dev machines, easy setup |
| systemd | Linux | Production servers |
| launchd | macOS | Mac dev machines |
| Docker + restart | Any | Containerized deployments |
| Cloudflare Durable Objects | Edge | Serverless persistence |

Read `references/daemon-implementations.md` for:
- PM2 ecosystem config for the agent
- systemd unit file template
- launchd plist for macOS
- Docker compose with health checks and restart
- State persistence strategies (SQLite, Redis, file-based)

## Daemon Capabilities

### Always-On Monitoring
- Watch git repos for pushes → auto-run tests
- Monitor error tracking (Sentry) → auto-triage
- Check deployment health → auto-rollback
- Scan dependencies → auto-PR for vulnerabilities

### Event Processing Loop
```
while (running) {
  events = poll_event_sources()  // or await webhook
  for event in events:
    route_to_handler(event)      // orchestrator routing
    log_event(event, result)     // structured logging
  sleep(interval)                // configurable polling interval
}
```

### Graceful Shutdown
1. Stop accepting new events
2. Finish processing current event
3. Flush logs and save state
4. Exit cleanly

## State Persistence Protocol

Read `references/daemon-implementations.md` for:
- Checkpoint state to disk every N minutes
- Resume from last checkpoint on restart
- Handle concurrent access (file locks, SQLite WAL)

## Integration with Other Skills
- **webhooks**: Daemon hosts the webhook receiver
- **scheduling**: Daemon executes cron jobs
- **loop-engine**: Daemon can run loops in background
- **logging**: All daemon activity is logged
- **semantic-memory**: Daemon can index new content continuously