---
name: multi-platform
description: "Multi-platform integration — connect agent to Slack, Discord, Telegram, WhatsApp, email, and other messaging platforms. Use for 'send to Slack', 'notify on Discord', 'Telegram bot', 'multi-channel', 'integrate with messaging'. Do NOT use for in-session chat or simple notifications."
---

# Multi-Platform Integration

## When to Activate
- User wants agent accessible from multiple platforms
- Need to send notifications to Slack, Discord, Telegram, email
- Building a bot that responds on messaging platforms
- Connecting workflows across communication channels

## Supported Platforms

| Platform | Integration Type | Key API |
|----------|-----------------|---------|
| Slack | Bot + Webhooks | Bolt SDK / Webhooks |
| Discord | Bot | discord.js |
| Telegram | Bot | Telegraf / node-telegram-bot-api |
| WhatsApp | Business API | Twilio / Meta Cloud API |
| Email | SMTP + IMAP | Nodemailer / IMAP |
| iMessage | AppleScript (Mac) | osascript bridge |
| Teams | Bot Framework | Microsoft Bot SDK |
| SMS | API | Twilio |

## Architecture Pattern

```
User message (any platform)
  ↓
Platform Adapter (normalize format)
  ↓
Message Router (parse intent)
  ↓
Orchestrator (route to skill)
  ↓
Execute + Generate Response
  ↓
Platform Adapter (format for platform)
  ↓
Send response back to user
```

## Platform Adapter Protocol

Each platform needs an adapter that:
1. **Receives** — Listen for messages/events
2. **Normalizes** — Convert to standard format
3. **Routes** — Pass to orchestrator
4. **Formats** — Convert response for platform (markdown → Slack blocks, etc.)
5. **Sends** — Deliver response back

### Standard Message Format
```typescript
interface NormalizedMessage {
  platform: 'slack' | 'discord' | 'telegram' | 'email' | 'whatsapp';
  user_id: string;
  channel_id: string;
  text: string;
  attachments?: Attachment[];
  reply_to?: string;
  metadata: Record<string, unknown>;
}
```

Read `references/platform-adapters.md` for:
- Slack bot setup with Bolt SDK
- Discord bot with discord.js
- Telegram bot with Telegraf
- Email integration with Nodemailer + IMAP
- Unified notification system across all platforms

## Notification Patterns

### Priority-Based Routing
```
CRITICAL (deploy failure, security alert) → Slack + SMS + Email
HIGH (test failures, PR reviews) → Slack + Email
MEDIUM (status updates, reports) → Slack only
LOW (info, metrics) → Email digest
```

## Integration with Other Skills
- **webhooks**: Platforms send events via webhooks
- **daemon-mode**: Daemon hosts platform bots
- **scheduling**: Scheduled messages across platforms
- **logging**: All cross-platform messages logged
- **orchestrator**: Messages from any platform route through orchestrator