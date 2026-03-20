---
description: Project status dashboard — git, tasks, health
allowed-tools: Read, Grep, Glob, Bash
---

Show a project status dashboard.

**Gather information:**
1. Git status: branch, recent commits, uncommitted changes
2. Package info: name, version, dependencies count
3. Test status: run tests if quick (< 30 seconds), otherwise show last known result
4. Build status: verify build passes
5. Active tasks: check TodoList or TASKS.md if they exist
6. Memory: read CLAUDE.md for project context

**Output format:**
```
## Project Status

**Project**: [name] v[version]
**Branch**: [branch] ([X commits ahead of main])
**Stack**: [detected stack]

### Health
- Build: ✅ Passing / ❌ Failing
- Tests: ✅ X passing, Y failing / ⚠️ No tests found
- Security: ✅ Clean / ⚠️ X vulnerabilities
- Dependencies: X total, Y outdated

### Recent Activity
- [latest 3-5 commits]

### Uncommitted Changes
- [modified files]

### Active Tasks
- [from TodoList or TASKS.md]
```
