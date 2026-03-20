# Long-Running Task & Context Management Patterns

## The Context Exhaustion Problem

Claude Code has a context window limit. On complex multi-step tasks (building an entire feature, large refactors, multi-file migrations), you can run out of context mid-task. These patterns prevent that.

## Stop-Hook Pattern (Ralph Wiggum Pattern)

For tasks that span many files or steps, use a checkpoint-and-resume approach:

### How It Works
```
1. Break the large task into discrete checkpoints
2. At each checkpoint, write progress to a file (PROGRESS.md)
3. If context is getting long, stop and ask for a fresh session
4. In the new session, read PROGRESS.md to resume exactly where you left off
```

### Progress File Template
```markdown
<!-- PROGRESS.md — auto-generated, do not edit manually -->
# Task: [Description]
Started: [timestamp]
Last checkpoint: [timestamp]

## Status: IN_PROGRESS

## Completed Steps
- [x] Step 1: Created database schema (files: src/db/schema.ts)
- [x] Step 2: Generated migration (files: drizzle/0001_create_tables.sql)
- [x] Step 3: Built API routes (files: src/app/api/users/route.ts, src/app/api/posts/route.ts)

## Current Step
- [ ] Step 4: Build frontend components
  - Next: Create UserProfile component at src/components/user-profile.tsx
  - Context: Uses the GET /api/users/:id endpoint, display name + avatar + posts list

## Remaining Steps
- [ ] Step 5: Add authentication middleware
- [ ] Step 6: Write tests
- [ ] Step 7: Update documentation

## Key Decisions Made
- Using Drizzle ORM with Neon Postgres
- UUID primary keys on all tables
- Server components for data fetching, client components only for interactivity

## Files Modified
- src/db/schema.ts (created)
- drizzle/0001_create_tables.sql (created)
- src/app/api/users/route.ts (created)
- src/app/api/posts/route.ts (created)
```

### When to Checkpoint
- After completing each major step (schema, API, components, tests)
- Before starting a step that will touch many files
- When the conversation is getting long (you can feel the context pressure)
- After making important architectural decisions

### Resume Protocol
When starting a new session after a checkpoint:
```
1. Read PROGRESS.md
2. Read CLAUDE.md for project context
3. Read the files listed in "Current Step" context
4. Continue from exactly where you left off
5. Update PROGRESS.md as you complete steps
```

## Task Decomposition Strategy

### For Large Features
```
Feature: "Add team management"

Phase 1: Data Layer (session 1)
  - Schema: teams, team_members, invitations tables
  - Migrations
  - Database queries / ORM methods
  → Checkpoint

Phase 2: API Layer (session 1 or 2)
  - CRUD routes for teams
  - Invitation flow routes
  - Auth middleware for team-scoped access
  → Checkpoint

Phase 3: Frontend (session 2 or 3)
  - Team settings page
  - Member management UI
  - Invitation flow UI
  → Checkpoint

Phase 4: Polish (session 3)
  - Tests
  - Error handling edge cases
  - Documentation
  → Done, clean up PROGRESS.md
```

### For Large Refactors
```
Refactor: "Migrate from Pages Router to App Router"

Batch 1: Infrastructure
  - Create app/ directory structure
  - Move layout to app/layout.tsx
  - Set up providers
  → Checkpoint

Batch 2: Static pages (low risk)
  - About, pricing, legal pages
  → Checkpoint

Batch 3: Dynamic pages
  - Dashboard, settings, profile
  - Convert getServerSideProps to server components
  → Checkpoint

Batch 4: API routes
  - Move pages/api/* to app/api/*/route.ts
  → Checkpoint

Batch 5: Cleanup
  - Remove old pages/ directory
  - Update imports
  - Run tests
  → Done
```

## Agent Delegation for Parallel Work

When multiple independent sub-tasks exist, spawn agents instead of doing them sequentially:

```
Task: "Set up the project with auth, database, and CI"

Instead of doing 3 things sequentially (uses lots of context):
→ Spawn agent: "Set up Drizzle database schema and migrations"
→ Spawn agent: "Configure NextAuth with Google and GitHub providers"
→ Spawn agent: "Create GitHub Actions CI pipeline"

Each agent gets a fresh context window.
Results are reported back to the orchestrator.
```

### When to Delegate vs. Do Inline
| Do Inline | Delegate to Agent |
|-----------|------------------|
| Quick edits (< 5 files) | Large feature implementation |
| Tasks needing conversation context | Independent sub-tasks |
| Interactive/iterative work | Boilerplate generation |
| Debugging (needs back-and-forth) | Code review, security scan |

## Compact Mode

For sessions that are getting long but don't need a full restart:

1. **Summarize and compress**: Before continuing, summarize what's been done so far in a concise note
2. **Use /compact**: Claude Code's built-in compaction command
3. **Offload to files**: Write intermediate results to files instead of keeping them in conversation context
4. **Reference, don't repeat**: "See the implementation in src/api/users.ts" instead of discussing the code again

## Cleanup

When a long-running task is complete:
```
1. Delete PROGRESS.md (or archive it)
2. Update CLAUDE.md with new patterns/decisions learned
3. Update memory/ with anything worth remembering
4. Run the self-eval skill to capture quality metrics
```
