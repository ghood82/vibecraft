# Learning Patterns — Reference

Pattern detection templates, adaptation decision trees, and inter-session correlation.

## Learning Log File Structure

Create `memory/learning/patterns.md` in the project root (or global memory if cross-project):

```
memory/
  learning/
    patterns.md          ← active pattern registry
    resolved/
      YYYY-MM-pattern.md ← archived resolved patterns
    sessions/
      YYYY-MM-DD.md      ← raw session observations
```

## Session Observation Template

At the end of each session, write to `memory/learning/sessions/YYYY-MM-DD.md`:

```markdown
# Session: [YYYY-MM-DD]

## Summary
- Tasks completed: [count]
- User corrections: [count]
- Loop iterations (avg): [n]
- Self-eval average score: [n/5]

## Observations

### Failures / Corrections
| What Happened | Correction Applied | Possible Pattern |
|--------------|-------------------|-----------------|
| [event] | [what user said/fixed] | [pattern name if recurring] |

### Wins (first-try successes worth reusing)
| What Worked | Context | Reusable? |
|-------------|---------|-----------|
| [approach] | [when/where] | YES/NO |

### Friction Points
| Step | Friction | Category |
|------|---------|----------|
| [step] | [what was slow/wrong] | routing/template/default/heuristic |

## Emerging Patterns
[Any issue that appeared here and also appeared in a prior session]

## Adaptations to Consider
[Specific proposed changes — wait for recurrence before applying]
```

## Pattern Registry Template

`memory/learning/patterns.md`:

```markdown
# Pattern Registry

Last updated: [YYYY-MM-DD]
Active patterns: [count]
Resolved this quarter: [count]

---

## ACTIVE PATTERNS

### P001: [Pattern Name]
**Type**: Failure | Win | Correction | Routing-miss
**Status**: PENDING | ADAPTING | VERIFYING | RESOLVED
**Sessions**: [count seen] | First: [date] | Last: [date]
**Priority**: HIGH | MEDIUM | LOW

**What happens**: [description]
**Trigger**: [what causes it]
**Impact**: [cost — rework, corrections, broken builds, lost time]

**Root cause**: [why]

**Adaptation**:
- File: [which file was changed]
- Change: [what was added/modified]
- Applied: [date]

**Verification**:
- Target: [what "resolved" looks like]
- Check-in date: [when to verify]
- Result: [PENDING | improved | unchanged | worse]

---

## RESOLVED PATTERNS

### P000: [Old Pattern Name] ✓
**Resolved**: [date] | **Sessions to resolve**: [n]
**What changed**: [summary of adaptation that fixed it]
```

## Worked Example Patterns

### Example 1 — Routing Miss

```markdown
### P003: Stripe questions routed to database skill
**Type**: Routing-miss
**Sessions**: 3 | First: 2025-01-12 | Last: 2025-01-19
**Priority**: MEDIUM

**What happens**: When user asks about payments or checkout, orchestrator routes to `database`
because Stripe stores data. But the user wants API integration help, not schema design.

**Root cause**: Orchestrator routing table maps "database" keyword before "api-design (stripe ref)"
for payment queries.

**Adaptation**:
- File: skills/orchestrator/SKILL.md (routing table)
- Change: Added "payments", "Stripe", "checkout", "billing" as explicit api-design (stripe ref) signals
  BEFORE database keyword match
- Applied: 2025-01-20

**Verification**:
- Target: Payment questions route to api-design with stripe reference, not database
- Result: RESOLVED after 2 sessions
```

### Example 2 — Recurring Failure

```markdown
### P007: Tests pass but deploy fails — missing env vars
**Type**: Failure
**Sessions**: 4 | First: 2025-02-03 | Last: 2025-02-24
**Priority**: HIGH

**What happens**: After green CI, production deploy fails because env vars present in CI
aren't set in the deployment environment. Tests mock them, so CI doesn't catch it.

**Root cause**: No env var checklist enforced before deploy. Tests mock env vars, hiding gaps.

**Adaptation**:
- File: skills/deployment/references/vercel.md
- Change: Added "Pre-Deploy Env Var Checklist" section — cross-check .env.example against
  deployment environment before promoting to production
- File: skills/env-secrets/SKILL.md
- Change: Added explicit warning: "Always verify deployment env matches .env.example before ship"
- Applied: 2025-02-25

**Verification**:
- Target: Zero deploy failures caused by missing env vars
- Result: VERIFYING (1 session without recurrence)
```

### Example 3 — Win to Replicate

```markdown
### P011: Parallel agent spawning for independent tasks (WIN)
**Type**: Win
**Sessions**: 5 | First: 2025-03-01
**Priority**: LOW (replicate, not fix)

**What happens**: When spawning code-reviewer + security-scanner in parallel (not sequential),
full review cycle is 2x faster with same quality.

**Root cause**: Tasks are fully independent — no shared context needed.

**Adaptation**:
- File: skills/orchestrator/SKILL.md
- Change: Added "Parallel execution is default for review + security scan" to workflow section
- Applied: 2025-03-05

**Verification**:
- Target: Review+security always run in parallel, not sequential
- Result: RESOLVED — consistently parallel now
```

## Adaptation Decision Tree

```
Is this the first occurrence?
  YES → Log as observation. Watch for recurrence. Do NOT adapt yet.
  NO  → Is it the 2nd occurrence?
          YES → Classify the pattern. Propose adaptation. Apply if confidence is high.
          NO  → Is it the 3rd+?
                  YES → Adapt immediately. Escalate if adaptation hasn't helped after 2 sessions.
```

## Inter-Session Pattern Correlation

Look for these correlation signals:

| Pattern A | Pattern B | Combined Signal |
|-----------|-----------|----------------|
| High loop count (>4 iter) | User correction on same area | Systemic issue — adapt the template |
| Same skill repeatedly wrong | User reroutes manually | Routing rule needs update |
| Self-eval score <3 on same dimension | Recurring failure type | Skill depth insufficient |
| Win reused 3+ times | Not documented | Should become default approach |

## When to Escalate vs. Adapt Locally

**Adapt locally** (update skill file):
- Pattern caused by wrong default in a specific skill
- Template scaffolds something that always needs manual correction
- Routing consistently wrong for a specific request type

**Escalate** (requires structural change):
- Pattern caused by fundamental ambiguity between two skills (boundary issue)
- New capability needed that no existing skill covers
- Pattern is cross-project (affects multiple repos/contexts)

For escalations: document in `memory/learning/patterns.md` with tag `ESCALATE`, then discuss with user.

## Learning Metrics

Track these to measure learning effectiveness:

| Metric | Baseline | Target |
|--------|---------|--------|
| Avg user corrections / session | [measure] | -50% after 10 sessions |
| Avg loop iterations | [measure] | -30% after 10 sessions |
| Routing accuracy (right first try) | [measure] | >90% |
| Patterns resolved / quarter | [measure] | >75% of new patterns |
| Days to adapt a pattern | [measure] | <7 days from 2nd occurrence |
