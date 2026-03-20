# Workflow Templates — Detailed Pipeline Definitions

## Pipeline Execution Rules

### Parallel Spawning
When agents are independent, spawn them in a single Agent tool call block:
```
→ Agent(code-reviewer, "Review all source files for quality issues")
→ Agent(security-scanner, "Full OWASP scan of the codebase")
→ Agent(qa-tester, "Generate and run comprehensive tests")
```
All three run concurrently. Wait for all to complete before proceeding.

### Sequential Chaining
When one agent's output feeds the next, wait for completion:
```
→ Agent(security-scanner, "Scan for vulnerabilities") → wait
→ [Apply fixes based on findings]
→ Agent(code-reviewer, "Verify the security patches are clean") → wait
→ Agent(qa-tester, "Run regression tests")
```

### Handoff Protocol
Each agent includes a "Handoff Recommendations" section in its output. Parse these to determine the next step:
- If an agent recommends another agent → spawn it automatically
- If an agent recommends a skill → route to the skill
- If no handoffs → workflow is complete
---

## Template 1: PR Review Pipeline

**Goal**: Comprehensive pre-merge quality gate
**Duration**: 3-8 minutes depending on codebase size

### Steps
1. **Parallel phase** — Spawn simultaneously:
   - `code-reviewer`: "Review all changed files for code quality, correctness, and maintainability"
   - `security-scanner`: "Scan changed files for security vulnerabilities and secrets"
2. **Merge phase** — Combine findings from both agents
3. **Conditional phase** — If either agent recommends testing:
   - `qa-tester`: "Generate tests for the areas flagged by code review and security scan"
4. **Report** — Unified PR review with go/no-go

### Success Criteria
- Zero CRIT/HIGH findings, or all have documented fixes
- No secrets detected
- Test coverage adequate for changed code

---

## Template 2: Ship-Ready Check

**Goal**: Full pre-deployment quality gate
**Duration**: 5-15 minutes

### Steps
1. **Fan-out phase** — Spawn all three simultaneously:
   - `code-reviewer`: "Full codebase review — focus on production readiness"
   - `security-scanner`: "Pre-deployment security scan with dependency audit"
   - `qa-tester`: "Run full test suite, generate missing tests for critical paths"
2. **Conditional phase** — If UI components exist:
   - `ux-auditor`: "Audit user-facing pages for accessibility and usability"
3. **Compile** — Unified ship-readiness report with GO/NO-GO
### Success Criteria
- All tests passing
- No CRIT/HIGH security findings
- No CRIT UX/accessibility findings
- Code review score: PASS or PASS WITH WARNINGS

---

## Template 3: Full Audit

**Goal**: Comprehensive analysis across all dimensions
**Duration**: 10-20 minutes

### Steps
1. **Full fan-out** — Spawn all 5 agents simultaneously:
   - `code-reviewer`: "Complete codebase audit for quality and maintainability"
   - `security-scanner`: "Full OWASP scan with dependency audit and secret detection"
   - `qa-tester`: "Audit test coverage, generate missing tests, run full suite"
   - `researcher`: "Evaluate tech stack currency — are dependencies up to date? Any better alternatives?"
   - `ux-auditor`: "Full UX and accessibility audit of all user-facing pages"
2. **Cross-reference phase** — Identify findings that span multiple domains
3. **Compile** — Unified audit report with prioritized action items

---

## Template 4: New Feature Pipeline

**Goal**: Build a feature with full quality assurance
**Duration**: 15-30 minutes

### Steps
1. `researcher`: "Evaluate approaches for [feature]. Consider [constraints]."
2. Build phase — orchestrator delegates to vibe-coding skill
3. `code-reviewer`: "Review the new feature implementation"
4. `security-scanner`: "Scan the new feature code for vulnerabilities"
5. `qa-tester`: "Generate comprehensive tests for the new feature"
6. If UI involved → `ux-auditor`: "Audit the new UI for usability and accessibility"

---

## Template 5: Security Remediation

**Goal**: Find and fix all security issues with verification
**Duration**: 10-25 minutes

### Steps
1. `security-scanner`: "Full vulnerability scan — find everything"
2. Triage findings by severity — fix CRIT and HIGH first
3. `code-reviewer`: "Verify security patches are correct and don't break logic"
4. `qa-tester`: "Generate regression tests for each patched vulnerability"
5. `security-scanner`: "Re-scan to confirm all issues are resolved"

### Success Criteria
- Zero CRIT/HIGH findings on re-scan
- All patches pass code review
- Regression tests exist for each fix

---

## Template 6: Accessibility Overhaul

**Goal**: Achieve WCAG 2.1 AA compliance
**Duration**: 10-20 minutes

### Steps
1. `ux-auditor`: "Full WCAG 2.1 AA audit of all user-facing pages"
2. Apply fixes for all 🔴 Critical findings first, then 🟡 Moderate
3. `code-reviewer`: "Verify semantic HTML changes and ARIA implementation"
4. `qa-tester`: "Generate automated accessibility tests using axe-core or similar"
5. `ux-auditor`: "Re-audit to confirm compliance"

### Success Criteria
- All 🔴 Critical accessibility issues resolved
- Color contrast meets 4.5:1 minimum
- All interactive elements keyboard accessible
- Automated a11y tests passing