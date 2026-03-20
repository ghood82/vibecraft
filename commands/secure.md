---
description: Security scan — vulnerabilities, secrets, hardening
allowed-tools: Read, Grep, Glob, Bash
argument-hint: [file or directory]
---

Run a comprehensive security scan.

Read the security skill at `${CLAUDE_PLUGIN_ROOT}/skills/security/SKILL.md`.
Reference OWASP patterns at `${CLAUDE_PLUGIN_ROOT}/skills/security/references/owasp-top-10.md`.
Use the checklist at `${CLAUDE_PLUGIN_ROOT}/skills/security/references/security-checklist.md`.

**Scan scope:** $ARGUMENTS (or entire project if not specified)

**Scan process:**
1. Dependency audit — `npm audit` or equivalent
2. Secret detection — grep for API keys, tokens, passwords
3. Code analysis — OWASP Top 10 patterns
4. Configuration review — headers, CORS, CSP
5. Auth review — if auth code exists

**Output:** Security scan report with findings by severity, impact assessment, and specific fixes.
Offer to fix Critical and High findings immediately.
