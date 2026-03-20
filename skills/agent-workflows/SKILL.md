---
name: agent-workflows
description: "Pre-built multi-agent workflow pipelines for comprehensive analysis. Coordinates code-reviewer, security-scanner, qa-tester, researcher, and ux-auditor agents in parallel and sequential patterns."
---

# Agent Workflows — Multi-Agent Pipelines

Orchestrate multiple VibeCraft agents into coordinated workflows for comprehensive project analysis.

## WHEN to Use This Skill
- "full review", "full audit", "audit everything", "review everything"
- "ship check", "ship-ready check", "is this ready to ship?"
- "PR review pipeline", "pre-merge check"
- "deep analysis", "thorough analysis", "comprehensive review"
- Any request that implies multiple agents working together

## WHEN NOT to Use This Skill
- Single-domain quick checks → use individual skills instead
- Quick code tip → use code-quality skill
- Quick security question → use security skill
- One-off test → use code-quality skill

## Available Agents

| Agent | Specialty | Color |
|-------|-----------|-------|
| **code-reviewer** | Multi-file code quality, correctness, maintainability | Blue |
| **security-scanner** | OWASP vulnerabilities, dependency audit, secret detection | Red |
| **qa-tester** | Test generation, execution, failure diagnosis | Green |
| **researcher** | Technology evaluation, multi-source comparison | Cyan |
| **ux-auditor** | UX evaluation, WCAG accessibility, design review | Magenta |
## Workflow Templates

### 1. PR Review Pipeline
**Trigger**: "review this PR", "pre-merge check"
**Pattern**: Parallel → merge results
- Spawn `code-reviewer` + `security-scanner` simultaneously
- Merge findings into a unified report
- If either recommends tests → spawn `qa-tester` as follow-up

### 2. Ship-Ready Check
**Trigger**: "is this ready to ship?", "ship check", "pre-deploy audit"
**Pattern**: Parallel fan-out → sequential follow-up
- Spawn `code-reviewer` + `security-scanner` + `qa-tester` in parallel
- Review combined results
- If UI components are in scope → spawn `ux-auditor` as follow-up
- Produce a go/no-go recommendation

### 3. Full Audit
**Trigger**: "audit everything", "full review", "comprehensive analysis"
**Pattern**: All agents in parallel
- Spawn all 5 agents simultaneously (code-reviewer, security-scanner, qa-tester, researcher, ux-auditor)
- Each agent scopes to its domain
- Compile a unified audit report with cross-cutting findings

### 4. New Feature Pipeline
**Trigger**: "build and verify this feature", "implement with full checks"
**Pattern**: Sequential pipeline
1. `researcher` — evaluate best approach
2. Build the feature (vibe-coding skill)
3. `code-reviewer` — review implementation
4. `security-scanner` — check for vulnerabilities
5. `qa-tester` — generate and run tests

### 5. Security Remediation
**Trigger**: "fix all security issues", "harden this codebase"
**Pattern**: Scan → fix → verify
1. `security-scanner` — find all vulnerabilities
2. Apply fixes (orchestrator coordinates)
3. `code-reviewer` — verify patches are clean
4. `qa-tester` — run regression tests
5. `security-scanner` — re-scan to confirm resolution
### 6. Accessibility Overhaul
**Trigger**: "make this accessible", "WCAG compliance", "accessibility overhaul"
**Pattern**: Audit → fix → verify
1. `ux-auditor` — full WCAG audit
2. Apply fixes (orchestrator coordinates)
3. `code-reviewer` — verify semantic HTML changes
4. `qa-tester` — generate a11y regression tests

## How to Execute a Workflow

1. Identify the workflow from the user's intent (or ask if ambiguous)
2. Read `references/workflow-templates.md` for detailed pipeline definitions
3. Spawn agents using the Agent tool — launch parallel agents in the same message
4. Collect results from each agent
5. Follow handoff recommendations from agent outputs
6. Compile a unified summary for the user

## Result Compilation

After all agents complete, produce a unified report:
```
## Workflow Report: [Pipeline Name]

### Agents Executed
[list of agents and their status]

### Combined Findings
[merged findings sorted by severity across all agents]

### Cross-Cutting Concerns
[issues that span multiple agent domains]

### Final Recommendation
GO / NO-GO with reasoning
```

See `references/workflow-templates.md` for detailed pipeline definitions and configuration.