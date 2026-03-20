---
description: Deploy to production — Vercel, Cloudflare, or any platform
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
argument-hint: [platform]
---

Ship the current project to production.

Read the deployment skill at `${CLAUDE_PLUGIN_ROOT}/skills/deployment/SKILL.md`.

**Pre-flight checklist (run automatically):**
1. Verify build passes
2. Run tests (if they exist)
3. Quick security scan for hardcoded secrets
4. Check environment variables are configured
5. Confirm the deployment target

**Deployment flow:**
1. Detect platform from config files or $ARGUMENTS
2. If Vercel: read `${CLAUDE_PLUGIN_ROOT}/skills/deployment/references/vercel.md`
3. If Cloudflare: read `${CLAUDE_PLUGIN_ROOT}/skills/deployment/references/cloudflare.md`
4. Run the pre-deployment checklist
5. Execute the deployment command
6. Verify the deployment succeeded (check URL, run smoke test)
7. Report: "Deployed. Live at [url]"

If any pre-flight check fails, report the issue and offer to fix it before deploying.
