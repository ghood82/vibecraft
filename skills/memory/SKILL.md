---
name: memory
description: "Persistent memory — remember stack, preferences, project context across sessions. Use when asked to remember, recall preferences, store context, my style, what stack do I use, project history. Do NOT use for codebase understanding (use codebase-understanding) or documentation (use docs-productivity)."
---

# Memory: Persistent Context & Preference Management

Store and recall your coding vibe across sessions—tech stack, style, preferences, and project context. Two-tier system: CLAUDE.md (hot cache) + memory/ (deep storage).

## Why Memory Matters

Context switching kills flow. Memory eliminates this by caching your tech stack, coding style, project continuity, smart defaults, and cross-session learning patterns.

## Two-Tier Architecture

### Tier 1: Hot Cache (CLAUDE.md)
A single, compact file in project root containing: Me, Stack Preferences, Coding Style, Active Projects, Terms & Shortcuts, Deep Memory Pointer. Read `references/memory-patterns.md` for the complete template.

### Tier 2: Deep Storage (memory/ Directory)
Organized folders: memory/stack/, memory/projects/, memory/decisions/, memory/team/, memory/patterns/. Detailed context files (1-3 pages each).

## What to Remember

### Always Remember
- Tech Stack (languages, frameworks, versions, why chosen)
- Coding Style (formatting rules, naming conventions, patterns)
- Project Context (state, key files, dependencies)
- Team Info (members, conventions, style)
- Past Decisions (architectures chosen, what was rejected)
- Common Patterns (reusable solutions)
- Preferences (shortcuts, IDE setup, testing approach)

### Smart Filtering (What NOT to Remember)
- Temporary debugging sessions (unless revealing a bug pattern)
- One-off scripts or experiments
- Publicly available documentation
- Frequently changing data (commit SHAs, API keys, credentials)
- Long stack traces (store the pattern, not the trace)

## Auto-Detection: Learning Your Preferences

Claude observes patterns and proposes memory updates. Detection heuristics: Stack Detection (if using tool in 3+ files, remember it), Style Detection (consistent patterns), Tool Detection (recurring commands), Pattern Detection (repeated implementations).

## Memory Lookup Flow

When Claude needs context:
1. Is this in CLAUDE.md? → Use immediately
2. If not: Check memory/ → Use if found
3. If not found: Ask user or derive from code

## Memory Promotion & Demotion

Frequently accessed (5+ times) → CLAUDE.md.
Used occasionally (1-2 times/month) → memory/ subdirectories.
Unused 6+ months → memory/archive/.

## Project-Specific Memory

Each repository maintains its own CLAUDE.md at repo root for project-specific overrides. Claude checks: Local CLAUDE.md → User's global CLAUDE.md → memory/.

## Cross-Session Learning

Earlier sessions inform later ones:
- Session 1: User builds pattern → Claude proposes storing it
- Session 2: Claude offers similar pattern → User confirms or modifies
- Session 3: Pattern applied automatically

## How to Use This Skill

### Initial Setup
1. Copy CLAUDE.md template to project root
2. Fill in Me, Stack Preferences, Coding Style
3. Create memory/ directory with subdirectories
4. Add detailed docs for your stack

### Ongoing Use
Claude checks memory/ at session start, proposes updates when patterns detected, suggests promotion/demotion periodically, links to memory/ docs when providing context.

### Manual Updates
Update CLAUDE.md directly (immediate), add files to memory/ for deeper context, propose new patterns, archive unused memory.

## Memory Format Guidelines

CLAUDE.md: Compact, ~150 words. Read in one breath.
memory/ files: Detailed, 1-3 pages with examples and rationale.
Decisions: Use ADR (Architecture Decision Record) format.
Patterns: Include problem, solution, example, when to use.

## Privacy & Security

CLAUDE.md and memory/ are LOCAL—never shared. Exclude credentials and API keys. Use .gitignore: `CLAUDE.md`, `memory/`. Review periodically to remove sensitive info.

Read `references/memory-patterns.md` for templates, detailed architecture, and setup instructions.
