# Context Strategy Implementations

## Semantic Compression

```typescript
interface CompressedContext {
  summary: string;        // Natural language summary
  key_decisions: string[]; // Important choices made
  active_files: string[];  // Files currently being edited
  errors_seen: string[];   // Errors encountered (for loop-engine)
  token_count: number;     // Tokens used by this block
}

async function compressContext(messages: Message[], maxTokens: number): Promise<CompressedContext> {
  // Group messages into logical chunks
  const chunks = groupByTopic(messages);
  
  // Summarize each chunk
  const summaries = await Promise.all(
    chunks.map(chunk => summarizeChunk(chunk))
  );
  
  // Merge summaries, staying under token budget
  return mergeSummaries(summaries, maxTokens);
}

function groupByTopic(messages: Message[]): Message[][] {
  const groups: Message[][] = [[]];
  let currentTopic = '';
  
  for (const msg of messages) {
    const topic = detectTopic(msg);
    if (topic !== currentTopic && groups[groups.length - 1].length > 0) {
      groups.push([]);
      currentTopic = topic;
    }
    groups[groups.length - 1].push(msg);
  }
  
  return groups;
}
```

## Token Counting

```typescript
// Approximate token counting (for planning, not billing)
function estimateTokens(text: string): number {
  // GPT/Claude average: ~4 chars per token for English
  return Math.ceil(text.length / 4);
}

// More accurate using tiktoken
import { encoding_for_model } from 'tiktoken';

function countTokens(text: string): number {
  const enc = encoding_for_model('gpt-4');
  const tokens = enc.encode(text);
  enc.free();
  return tokens.length;
}

class TokenBudget {
  private budget: number;
  private used: number = 0;
  
  constructor(totalBudget: number) {
    this.budget = totalBudget;
  }
  
  allocate(category: string, tokens: number): boolean {
    if (this.used + tokens > this.budget * 0.9) return false;
    this.used += tokens;
    return true;
  }
  
  get remaining(): number { return this.budget - this.used; }
  get utilizationPercent(): number { return (this.used / this.budget) * 100; }
  
  shouldCompact(): 'none' | 'warning' | 'auto' | 'emergency' {
    const pct = this.utilizationPercent;
    if (pct >= 95) return 'emergency';
    if (pct >= 85) return 'auto';
    if (pct >= 70) return 'warning';
    return 'none';
  }
}
```

## PROGRESS.md Handoff Format

When a session needs to hand off to a new session:

```markdown
# PROGRESS.md — Session Handoff

## Status: IN_PROGRESS
## Task: [what we're working on]
## Started: [ISO timestamp]
## Strategy: [which context strategy was in use]

## Current State
- Working on: [specific file/feature]
- Last action: [what was just done]
- Next step: [what to do next]

## Key Decisions Made
1. [Decision 1 — and why]
2. [Decision 2 — and why]

## Files Modified
- src/auth.ts — Added token validation middleware
- src/user.ts — Fixed import path
- tests/auth.test.ts — Updated to test new middleware

## Errors Still Open
- [ ] api.test.ts:88 — Timeout (needs await)

## Context for New Session
[Any important context the new session needs to know
that isn't obvious from the code itself]

## Patterns Learned This Session
- Import errors should be fixed before logic errors
- Auth tests flake in parallel — use --runInBand
```

## Dynamic Strategy Switching

```typescript
class ContextEngine {
  private strategy: ContextStrategy;
  private tokenBudget: TokenBudget;

  constructor(totalTokens: number = 200000) {
    this.tokenBudget = new TokenBudget(totalTokens);
    this.strategy = new SlidingWindowStrategy(); // default
  }

  async selectStrategy(taskType: string): Promise<void> {
    const strategies: Record<string, () => ContextStrategy> = {
      'quick-fix': () => new SlidingWindowStrategy(),
      'refactor': () => new SemanticCompressionStrategy(),
      'feature': () => new TaskFocusedStrategy(),
      'multi-session': () => new HierarchicalStrategy(),
    };

    const factory = strategies[taskType] || strategies['quick-fix'];
    this.strategy = factory();
  }

  async processContext(messages: Message[], files: string[]): Promise<ProcessedContext> {
    const compaction = this.tokenBudget.shouldCompact();
    
    if (compaction === 'emergency') {
      return this.emergencyCompact(messages, files);
    }
    if (compaction === 'auto') {
      return this.strategy.compact(messages, files, this.tokenBudget.remaining);
    }
    
    return this.strategy.process(messages, files);
  }

  private async emergencyCompact(messages: Message[], files: string[]): Promise<ProcessedContext> {
    // Keep only: system prompt, memory, last 3 messages, current file
    const essential = messages.slice(-3);
    const summary = await summarizeAll(messages.slice(0, -3));
    return { summary, messages: essential, files: files.slice(0, 1) };
  }
}
```