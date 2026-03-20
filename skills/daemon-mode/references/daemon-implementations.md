# Daemon Implementation Patterns

## PM2 (Recommended for Development)

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'vibe-agent',
    script: './src/daemon.ts',
    interpreter: 'npx',
    interpreter_args: 'tsx',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '500M',
    env: {
      NODE_ENV: 'production',
      LOG_LEVEL: 'info',
      POLL_INTERVAL: '30000',
      STATE_DIR: '.loop/daemon-state'
    },
    error_file: '.loop/logs/daemon-error.log',
    out_file: '.loop/logs/daemon-out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    max_restarts: 10,
    restart_delay: 5000,
  }]
};
```

```bash
# Start daemon
pm2 start ecosystem.config.js

# Monitor
pm2 monit

# Logs
pm2 logs vibe-agent

# Stop
pm2 stop vibe-agent

# Auto-start on boot
pm2 startup
pm2 save
```

## macOS launchd

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.vibe-coder.daemon</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/node</string>
        <string>/Users/garethhood/projects/vibe-agent/src/daemon.js</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
    </dict>
    <key>StandardOutPath</key>
    <string>/Users/garethhood/.loop/logs/daemon-out.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/garethhood/.loop/logs/daemon-error.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>NODE_ENV</key>
        <string>production</string>
    </dict>
    <key>ThrottleInterval</key>
    <integer>10</integer>
</dict>
</plist>
```

```bash
# Install
cp com.vibe-coder.daemon.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.vibe-coder.daemon.plist

# Check status
launchctl list | grep vibe-coder

# Stop
launchctl unload ~/Library/LaunchAgents/com.vibe-coder.daemon.plist
```

## Linux systemd

```ini
# /etc/systemd/system/vibe-agent.service
[Unit]
Description=Vibe Coder Agent Daemon
After=network.target

[Service]
Type=simple
User=deploy
WorkingDirectory=/opt/vibe-agent
ExecStart=/usr/bin/node src/daemon.js
Restart=always
RestartSec=5
Environment=NODE_ENV=production
Environment=LOG_LEVEL=info
StandardOutput=append:/var/log/vibe-agent/out.log
StandardError=append:/var/log/vibe-agent/error.log

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/vibe-agent/.loop

[Install]
WantedBy=multi-user.target
```

## Docker Compose

```yaml
version: '3.8'
services:
  vibe-agent:
    build: .
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - POLL_INTERVAL=30000
    volumes:
      - ./loop-state:/app/.loop
      - ./logs:/app/logs
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
```

## Core Daemon Script

```typescript
// src/daemon.ts
import { logger } from './logger';
import { EventLoop } from './event-loop';
import { StateStore } from './state-store';

const POLL_INTERVAL = parseInt(process.env.POLL_INTERVAL || '30000');
const STATE_DIR = process.env.STATE_DIR || '.loop/daemon-state';

class VibeDaemon {
  private running = false;
  private state: StateStore;
  private eventLoop: EventLoop;

  constructor() {
    this.state = new StateStore(STATE_DIR);
    this.eventLoop = new EventLoop();
  }

  async start() {
    logger.info('Daemon starting');
    this.running = true;

    // Restore state from last run
    await this.state.restore();

    // Register signal handlers
    process.on('SIGTERM', () => this.shutdown('SIGTERM'));
    process.on('SIGINT', () => this.shutdown('SIGINT'));

    // Main loop
    while (this.running) {
      try {
        const events = await this.eventLoop.poll();
        for (const event of events) {
          await this.processEvent(event);
        }
        await this.state.checkpoint();
      } catch (err) {
        logger.error('Event loop error', { error: String(err) });
      }
      await this.sleep(POLL_INTERVAL);
    }
  }

  async processEvent(event: any) {
    logger.info('Processing event', { type: event.type, source: event.source });
    // Route to appropriate handler via orchestrator logic
  }

  async shutdown(signal: string) {
    logger.info(`Shutdown requested: ${signal}`);
    this.running = false;
    await this.state.checkpoint();
    await this.state.close();
    logger.info('Daemon stopped cleanly');
    process.exit(0);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

const daemon = new VibeDaemon();
daemon.start().catch(err => {
  logger.fatal('Daemon crashed', { error: String(err) });
  process.exit(1);
});
```