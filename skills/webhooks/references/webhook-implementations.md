# Webhook Implementation Patterns

## GitHub Webhook Server (Express + TypeScript)

```typescript
import express from 'express';
import crypto from 'crypto';
import { logger } from './logger';

const app = express();
app.use(express.json());

const GITHUB_SECRET = process.env.GITHUB_WEBHOOK_SECRET!;

// Signature verification middleware
function verifyGitHubSignature(req: express.Request, res: express.Response, next: express.NextFunction) {
  const signature = req.headers['x-hub-signature-256'] as string;
  if (!signature) return res.status(401).json({ error: 'Missing signature' });

  const body = JSON.stringify(req.body);
  const expected = 'sha256=' + crypto.createHmac('sha256', GITHUB_SECRET).update(body).digest('hex');

  if (!crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected))) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  next();
}

// Event router
const handlers: Record<string, (payload: any) => Promise<void>> = {
  'push': async (payload) => {
    const branch = payload.ref.replace('refs/heads/', '');
    logger.info('Push event', { branch, commits: payload.commits.length });

    if (branch === 'main') {
      // Trigger test suite via loop-engine pattern
      await runTestFixLoop();
    }
  },

  'pull_request': async (payload) => {
    const { action, number, pull_request } = payload;
    logger.info('PR event', { action, number, title: pull_request.title });

    if (action === 'opened' || action === 'synchronize') {
      await runCodeReview(pull_request);
      await runSecurityScan(pull_request);
    }
  },

  'issues': async (payload) => {
    const { action, issue } = payload;
    if (action === 'opened' && issue.labels.some((l: any) => l.name === 'bug')) {
      await triageBugReport(issue);
    }
  },

  'release': async (payload) => {
    if (payload.action === 'published') {
      await triggerDeployment(payload.release.tag_name);
    }
  }
};

app.post('/webhook/github', verifyGitHubSignature, async (req, res) => {
  const event = req.headers['x-github-event'] as string;
  const deliveryId = req.headers['x-github-delivery'] as string;

  // Respond immediately
  res.status(200).json({ received: true, delivery: deliveryId });

  // Process async
  const handler = handlers[event];
  if (handler) {
    try {
      await handler(req.body);
    } catch (err) {
      logger.error('Webhook handler failed', { event, deliveryId, error: String(err) });
    }
  }
});
```

## Slack Bot (Bolt Framework)

```typescript
import { App } from '@slack/bolt';
import { logger } from './logger';

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  socketMode: true,
  appToken: process.env.SLACK_APP_TOKEN,
});

// Listen for messages mentioning the bot
app.event('app_mention', async ({ event, say }) => {
  logger.info('Bot mentioned', { user: event.user, text: event.text });
  
  const command = parseCommand(event.text);
  
  switch (command.action) {
    case 'deploy':
      await say(`Starting deployment to ${command.target}...`);
      await triggerDeployment(command.target);
      await say('Deployed successfully!');
      break;
    case 'status':
      const status = await getProjectStatus();
      await say(formatStatusReport(status));
      break;
    case 'test':
      await say('Running test suite...');
      const result = await runTests();
      await say(`Tests complete: ${result.passed}/${result.total} passed`);
      break;
  }
});

// Slash commands
app.command('/vibe', async ({ command, ack, respond }) => {
  await ack();
  await respond(`Starting vibe coding session: ${command.text}`);
  // Route to orchestrator
});

app.command('/ship', async ({ command, ack, respond }) => {
  await ack();
  await respond('Initiating deployment pipeline...');
  // Route to deployment skill
});

await app.start(3000);
```

## Cloudflare Worker Webhook Handler

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    if (request.method !== 'POST') {
      return new Response('Method not allowed', { status: 405 });
    }

    const url = new URL(request.url);
    const body = await request.text();

    // Route by path
    switch (url.pathname) {
      case '/webhook/github':
        return handleGitHub(request, body, env);
      case '/webhook/stripe':
        return handleStripe(request, body, env);
      case '/webhook/custom':
        return handleCustom(request, body, env);
      default:
        return new Response('Not found', { status: 404 });
    }
  }
};

async function handleGitHub(req: Request, body: string, env: Env): Promise<Response> {
  // Verify signature
  const signature = req.headers.get('x-hub-signature-256');
  const key = await crypto.subtle.importKey(
    'raw',
    new TextEncoder().encode(env.GITHUB_SECRET),
    { name: 'HMAC', hash: 'SHA-256' },
    false,
    ['sign']
  );
  const sig = await crypto.subtle.sign('HMAC', key, new TextEncoder().encode(body));
  const expected = 'sha256=' + Array.from(new Uint8Array(sig)).map(b => b.toString(16).padStart(2, '0')).join('');

  if (signature !== expected) {
    return new Response('Unauthorized', { status: 401 });
  }

  const event = req.headers.get('x-github-event');
  const payload = JSON.parse(body);

  // Store event in KV for processing
  await env.WEBHOOK_EVENTS.put(
    `github:${Date.now()}`,
    JSON.stringify({ event, payload }),
    { expirationTtl: 86400 }
  );

  return new Response(JSON.stringify({ received: true }), {
    headers: { 'Content-Type': 'application/json' }
  });
}
```

## Generic Webhook Receiver with Event Registry

```typescript
interface WebhookConfig {
  source: string;
  events: string[];
  secret: string;
  verifyFn: (req: Request, secret: string) => boolean;
  handler: (event: string, payload: any) => Promise<void>;
}

class WebhookRegistry {
  private configs: Map<string, WebhookConfig> = new Map();

  register(config: WebhookConfig) {
    this.configs.set(config.source, config);
  }

  async handle(source: string, event: string, req: Request, body: any): Promise<boolean> {
    const config = this.configs.get(source);
    if (!config) return false;
    if (!config.events.includes(event) && !config.events.includes('*')) return false;
    if (!config.verifyFn(req, config.secret)) throw new Error('Signature verification failed');
    
    await config.handler(event, body);
    return true;
  }
}

// Usage:
const registry = new WebhookRegistry();

registry.register({
  source: 'github',
  events: ['push', 'pull_request', 'issues'],
  secret: process.env.GITHUB_SECRET!,
  verifyFn: verifyGitHubSignature,
  handler: async (event, payload) => {
    // Route to appropriate skill
  }
});
```

## Security Best Practices

### HMAC Signature Verification
Always use `crypto.timingSafeEqual` to prevent timing attacks:
```typescript
function verifyHMAC(payload: string, signature: string, secret: string): boolean {
  const expected = crypto.createHmac('sha256', secret).update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
}
```

### Replay Protection
```typescript
const PROCESSED_EVENTS = new Set<string>();
const MAX_AGE_MS = 5 * 60 * 1000; // 5 minutes

function isReplay(deliveryId: string, timestamp: number): boolean {
  if (PROCESSED_EVENTS.has(deliveryId)) return true;
  if (Date.now() - timestamp > MAX_AGE_MS) return true;
  PROCESSED_EVENTS.add(deliveryId);
  // Cleanup old entries periodically
  return false;
}
```