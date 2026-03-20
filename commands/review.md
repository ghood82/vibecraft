---
description: Full code review — security, performance, correctness
allowed-tools: Read, Grep, Glob, Bash
argument-hint: [file or directory]
---

Conduct a comprehensive code review.

Read the code-quality skill at `${CLAUDE_PLUGIN_ROOT}/skills/code-quality/SKILL.md`.
Use the review checklist at `${CLAUDE_PLUGIN_ROOT}/skills/code-quality/references/review-checklist.md`.

**Scope:**
- If $ARGUMENTS specifies a file or directory, review that
- If no arguments, review recent changes: `git diff --name-only HEAD~1` or staged files
- For a full project review, scan all source files

**Review dimensions:**
1. Security — injection, auth, secrets, dependencies
2. Performance — queries, memory, bundle size, caching
3. Correctness — edge cases, error handling, types
4. Maintainability — naming, structure, duplication

**Output:** Findings grouped by severity with specific file:line references and actionable fixes.
Always include positive observations alongside issues.
