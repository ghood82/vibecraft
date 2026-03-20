---
name: context-engine
description: "Pluggable context management — swap context strategies without changing core code, manage token budgets, smart context windowing. Use for 'context too long', 'running out of context', 'manage context', 'context strategy'. Do NOT use for basic memory or file reading."
---

# Context Engine — Pluggable Context Management

## When to Activate
- Session context is getting large (approaching token limits)
- User says "context is too long", "summarize and continue"
- Need to switch context strategies for different task types
- Optimizing token usage for efficiency

## Context Strategies (Plug-and-Play)

### 1. Sliding Window (Default)
Keep the most recent N messages plus system context. Best for interactive coding.
```
[System] [Memory] [...last 20 messages...] [Current]
```

### 2. Semantic Compression
Summarize old context, keep recent verbatim. Best for long sessions.
```
[System] [Memory] [Summary of messages 1-50] [...last 10 messages...] [Current]
```

### 3. Task-Focused
Only keep context relevant to the current task. Best for deep work.
```
[System] [Memory] [Task-relevant files] [Task-relevant history] [Current]
```

### 4. Hierarchical
Multi-level summaries with increasing detail toward recent. Best for complex projects.
```
[System] [Memory] [Project summary] [Session summary] [...last 5 messages...] [Current]
```

## Token Budget Management

### Budget Allocation
```
Total context: 200K tokens
  ├── System prompt + skills:  ~15K (7.5%)
  ├── Memory + CLAUDE.md:      ~5K  (2.5%)
  ├── File contents:           ~80K (40%)
  ├── Conversation history:    ~80K (40%)
  └── Response headroom:       ~20K (10%)
```

### Auto-Compaction Triggers
- **Warning** at 70% usage: Suggest summarizing old context
- **Auto-compact** at 85%: Summarize oldest 50% of conversation
- **Emergency** at 95%: Aggressive compression, keep only essentials

## Strategy Selection Protocol

| Task Type | Best Strategy | Why |
|-----------|---------------|-----|
| Quick fix / bug | Sliding Window | Need recent context only |
| Long refactor | Semantic Compression | Hours of context, need key decisions |
| New feature build | Task-Focused | Deep in specific files |
| Multi-session project | Hierarchical | Need broad + deep context |

Read `references/context-strategies.md` for:
- Compression algorithm implementations
- Token counting utilities
- Context switching between strategies mid-session
- PROGRESS.md handoff format for session continuity

## Compact Mode Protocol

When context is exhausted:
1. Write PROGRESS.md with current state, decisions, next steps
2. Summarize all learned patterns → self-improvement
3. Save any new memory entries
4. Present user with handoff summary
5. New session reads PROGRESS.md to resume

## Integration with Other Skills
- **semantic-memory**: Context engine queries memory for relevant context
- **orchestrator**: Strategy selection based on task type
- **self-improvement**: Compression preserves learned patterns
- **logging**: Track token usage over time for optimization