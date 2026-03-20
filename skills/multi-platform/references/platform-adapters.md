# Platform Adapter Implementations

## Unified Notification System

```typescript
interface NotificationConfig {
  platform: string;
  channel: string;
  priority: 'critical' | 'high' | 'medium' | 'low';
}

class NotificationHub {
  private adapters: Map<string, PlatformAdapter> = new Map();
  private routes: NotificationConfig[] = [];

  register(platform: string, adapter: PlatformAdapter) {
    this.adapters.set(platform, adapter);
  }

  addRoute(config: NotificationConfig) {
    this.routes.push(config);
  }

  async notify(message: string, priority: string, metadata?: Record<string, unknown>) {
    const targets = this.routes.filter(r => r.priority === priority);
    await Promise.allSettled(
      targets.map(target => {
        const adapter = this.adapters.get(target.platform);
        return adapter?.send(target.channel, message, metadata);
      })
    );
  }
}
```

## Slack Adapter

```typescript
import { WebClient } from '@slack/web-api';

class SlackAdapter implements PlatformAdapter {
  private client: WebClient;

  constructor(token: string) {
    this.client = new WebClient(token);
  }

  async send(channel: string, message: string, metadata?: Record<string, unknown>) {
    // Convert markdown to Slack blocks
    const blocks = this.markdownToBlocks(message);
    await this.client.chat.postMessage({
      channel,
      text: message, // fallback
      blocks,
      ...(metadata?.thread_ts && { thread_ts: metadata.thread_ts as string })
    });
  }

  async listen(callback: (msg: NormalizedMessage) => void) {
    // Use Bolt SDK event listener — see webhooks skill
  }

  private markdownToBlocks(md: string): any[] {
    const blocks: any[] = [];
    const sections = md.split('\n\n');
    for (const section of sections) {
      if (section.startsWith('# ')) {
        blocks.push({ type: 'header', text: { type: 'plain_text', text: section.replace('# ', '') } });
      } else if (section.startsWith('```')) {
        blocks.push({ type: 'section', text: { type: 'mrkdwn', text: section } });
      } else {
        blocks.push({ type: 'section', text: { type: 'mrkdwn', text: section } });
      }
    }
    return blocks;
  }
}
```

## Discord Adapter

```typescript
import { Client, GatewayIntentBits, TextChannel } from 'discord.js';

class DiscordAdapter implements PlatformAdapter {
  private client: Client;

  constructor(token: string) {
    this.client = new Client({
      intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages, GatewayIntentBits.MessageContent]
    });
    this.client.login(token);
  }

  async send(channelId: string, message: string) {
    const channel = await this.client.channels.fetch(channelId) as TextChannel;
    // Discord has 2000 char limit — chunk if needed
    const chunks = this.chunkMessage(message, 2000);
    for (const chunk of chunks) {
      await channel.send(chunk);
    }
  }

  async listen(callback: (msg: NormalizedMessage) => void) {
    this.client.on('messageCreate', (msg) => {
      if (msg.author.bot) return;
      callback({
        platform: 'discord',
        user_id: msg.author.id,
        channel_id: msg.channelId,
        text: msg.content,
        metadata: { guild_id: msg.guildId }
      });
    });
  }

  private chunkMessage(text: string, maxLen: number): string[] {
    if (text.length <= maxLen) return [text];
    const chunks: string[] = [];
    let remaining = text;
    while (remaining.length > 0) {
      const breakPoint = remaining.lastIndexOf('\n', maxLen) || maxLen;
      chunks.push(remaining.slice(0, breakPoint));
      remaining = remaining.slice(breakPoint);
    }
    return chunks;
  }
}
```

## Telegram Adapter

```typescript
import { Telegraf } from 'telegraf';

class TelegramAdapter implements PlatformAdapter {
  private bot: Telegraf;

  constructor(token: string) {
    this.bot = new Telegraf(token);
  }

  async send(chatId: string, message: string) {
    // Telegram supports markdown natively
    await this.bot.telegram.sendMessage(chatId, message, { parse_mode: 'Markdown' });
  }

  async listen(callback: (msg: NormalizedMessage) => void) {
    this.bot.on('text', (ctx) => {
      callback({
        platform: 'telegram',
        user_id: String(ctx.from.id),
        channel_id: String(ctx.chat.id),
        text: ctx.message.text,
        metadata: { username: ctx.from.username }
      });
    });
    this.bot.launch();
  }

  stop() {
    this.bot.stop('SIGTERM');
  }
}
```

## Email Adapter

```typescript
import nodemailer from 'nodemailer';
import Imap from 'node-imap';

class EmailAdapter implements PlatformAdapter {
  private transporter: nodemailer.Transporter;

  constructor(config: { host: string; port: number; user: string; pass: string }) {
    this.transporter = nodemailer.createTransport({
      host: config.host,
      port: config.port,
      secure: config.port === 465,
      auth: { user: config.user, pass: config.pass }
    });
  }

  async send(to: string, message: string, metadata?: Record<string, unknown>) {
    await this.transporter.sendMail({
      from: metadata?.from as string || 'vibe-agent@example.com',
      to,
      subject: metadata?.subject as string || 'Vibe Agent Notification',
      text: message,
      html: this.markdownToHtml(message)
    });
  }

  private markdownToHtml(md: string): string {
    return md
      .replace(/^### (.*$)/gm, '<h3>$1</h3>')
      .replace(/^## (.*$)/gm, '<h2>$1</h2>')
      .replace(/^# (.*$)/gm, '<h1>$1</h1>')
      .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
      .replace(/\*(.*?)\*/g, '<em>$1</em>')
      .replace(/`(.*?)`/g, '<code>$1</code>')
      .replace(/\n/g, '<br>');
  }
}
```