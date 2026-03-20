---
description: Research a topic — compare, evaluate, recommend
allowed-tools: Read, Grep, Glob, WebSearch, WebFetch
argument-hint: [topic or question]
---

Research a technical topic and provide actionable recommendations.

Read the research skill at `${CLAUDE_PLUGIN_ROOT}/skills/research/SKILL.md`.

**Research question:** $ARGUMENTS

**Process:**
1. Clarify the question if ambiguous
2. Search for current, authoritative information
3. Compare options using a structured decision matrix
4. Verify claims across multiple sources
5. Deliver a clear recommendation with reasoning

**Output:** Structured comparison or recommendation report with sources cited.

Prefer official documentation and recent sources (< 12 months). Flag anything outdated.
