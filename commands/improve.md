---
description: Self-evaluate and improve — learn from this session
allowed-tools: Read, Write, Edit, Grep, Glob
---

Run a self-evaluation and improvement cycle.

Read the self-eval skill at `${CLAUDE_PLUGIN_ROOT}/skills/self-eval/SKILL.md`.
Read the memory skill at `${CLAUDE_PLUGIN_ROOT}/skills/memory/SKILL.md`.

**Process:**
1. Review what was accomplished this session
2. Evaluate quality across dimensions:
   - Correctness: Did the code work as intended?
   - Completeness: Were all requirements met?
   - Code quality: Clean, typed, tested?
   - Security: Any vulnerabilities introduced?
   - UX/DX: Is it pleasant to use and maintain?
3. Score each dimension 1-5
4. Identify what went well and what could improve
5. Update memory:
   - New preferences discovered
   - Patterns that worked well
   - Mistakes to avoid next time
6. Update CLAUDE.md if there are new insights worth caching

**Output:** Brief retrospective with scores, lessons learned, and memory updates made.
