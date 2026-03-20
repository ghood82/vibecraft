---
name: feature
description: "Structured feature development workflow — 7 phases from requirements to docs"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Agent
---

# Feature Development Workflow

Run through all 7 phases for building a complete feature. This is the structured alternative to `/vibe` — use when quality and completeness matter more than speed.

**Feature:** $ARGUMENTS

## Phase 1: Requirements

Clarify what "done" looks like before writing any code.

- What problem does this solve?
- Who is the user?
- What are the acceptance criteria?
- What are the edge cases?
- Are there design references (Figma, wireframes)?

Write a brief spec (3-5 bullet points for each: goal, user stories, acceptance criteria, out of scope).

## Phase 2: Codebase Exploration

Understand the existing code before changing it.

- Read the relevant files and directories
- Map the data flow for related features
- Identify where the new feature plugs in
- Check for existing patterns to follow (component structure, API patterns, naming)
- Note any tech debt that might affect the feature

## Phase 3: Architecture

Design the solution before implementing.

- What components/files need to be created or modified?
- What's the data model? (new tables, columns, relations)
- What's the API surface? (new endpoints, modified endpoints)
- Are there dependencies? (new packages, external APIs)
- What's the migration plan if changing existing data?

Create a TodoList with the implementation plan.

## Phase 4: Implementation

Build it, following the architecture from Phase 3.

Read `${CLAUDE_PLUGIN_ROOT}/skills/vibe-coding/SKILL.md` for coding standards.
Read `${CLAUDE_PLUGIN_ROOT}/skills/vibe-coding/references/coding-patterns.md` for patterns.

- Implement backend first (schema, API, business logic)
- Then frontend (components, pages, state)
- Follow existing project patterns
- Handle errors and edge cases
- Add loading states and empty states

## Phase 5: Testing

Verify it works correctly.

Read `${CLAUDE_PLUGIN_ROOT}/skills/code-quality/references/testing-strategies.md` for testing patterns.

- Write unit tests for business logic
- Write integration tests for API routes
- Manual testing of the happy path and edge cases
- Check accessibility (keyboard nav, screen reader)

## Phase 6: Review

Self-review before considering it done.

Read `${CLAUDE_PLUGIN_ROOT}/skills/code-quality/SKILL.md` for review standards.
Read `${CLAUDE_PLUGIN_ROOT}/skills/security/SKILL.md` for security checks.

- Code review: security, performance, correctness, maintainability
- Quick security scan: auth, input validation, secrets
- Performance check: no N+1 queries, no unnecessary re-renders

## Phase 7: Documentation

Document what was built.

- Update README if the feature is user-facing
- Add JSDoc/comments for complex logic
- Update API docs if endpoints changed
- Create a changelog entry
- Update memory with any new patterns or decisions
