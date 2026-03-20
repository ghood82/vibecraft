---
description: Start a vibe coding session — build, create, prototype
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
argument-hint: [what to build]
---

Enter vibe coding mode. The user wants to build something.

Read the orchestrator skill at `${CLAUDE_PLUGIN_ROOT}/skills/orchestrator/SKILL.md` for routing guidance.
Read the vibe-coding skill at `${CLAUDE_PLUGIN_ROOT}/skills/vibe-coding/SKILL.md` for coding patterns.

If CLAUDE.md exists in the project root, read it for preferences and context.

**Process:**
1. Understand what the user wants to build: $ARGUMENTS
2. Detect the project stack from existing config files (package.json, tsconfig.json, wrangler.toml, etc.)
3. If new project: scaffold using templates from the vibe-coding skill
4. If existing project: match existing patterns and conventions
5. Generate code that is typed, handles errors, and follows the project's style
6. Show progress with a TodoList for multi-step work
7. Verify the build works: run the build command or dev server

Keep momentum. Ship incrementally. Don't over-ask — detect and act.
