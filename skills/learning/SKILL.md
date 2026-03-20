---
name: learning
description: "Closed-loop self-improvement: track failures and successes across sessions, detect recurring patterns, adjust behavior automatically. Use for: learning from past mistakes, pattern recognition across sessions, feedback loops, behavior adjustment, what keeps going wrong, recurring issues. Do NOT use for single-task evaluation (use self-eval) or storing preferences (use memory)."
---

# Learning — Closed-Loop Self-Improvement

Self-eval grades one task. Memory stores preferences. Learning changes the system itself. This skill closes the feedback loop: observe patterns across sessions → detect what keeps failing → update the routing logic, templates, and defaults accordingly.

## When to Activate

- "Learn from what went wrong last session"
- "Why does this keep happening?"
- "Update yourself based on what worked"
- "I keep having to correct you on X"
- Any pattern that recurs across 2+ sessions

## The Learning Loop

```
OBSERVE → DETECT → ADAPT → VERIFY

1. OBSERVE: Collect outcomes from past sessions
   (self-eval scores, user corrections, failed loops, recurring friction)

2. DETECT: Find patterns — what keeps failing? What consistently works?
   (>= 2 occurrences = a pattern. 1 occurrence = noise.)

3. ADAPT: Update behavior — routing rules, templates, defaults, skill guidance
   (Document the adaptation explicitly so it persists)

4. VERIFY: In the next session, check if the adaptation helped
   (Did the pattern stop recurring? Did quality improve?)
```

## What to Observe

Collect from each session:
- **Failures**: What broke, needed rework, or required user correction
- **Wins**: Approaches that worked first time with no revision
- **User corrections**: "No, do it this way instead" — highest signal
- **Loop counts**: How many iterations a fix-loop needed (high count = systemic issue)
- **Routing misses**: Wrong skill activated for a request

## Pattern Detection

A pattern exists when the same issue appears 2+ times:

```
Failure Pattern: "Tests pass but integration breaks"
Sessions: 3
Root cause: Mocking external services too aggressively
Adaptation: Default to integration tests for service boundaries, unit tests for logic only

User Correction Pattern: "Don't use default exports"
Sessions: 4
Adaptation: Add named-exports-only rule to vibe-coding skill routing
```

Single-session issues are noise — log them but don't adapt yet. If they recur, they become patterns.

## Adaptation Types

| Type | What Changes | How |
|------|-------------|-----|
| **Routing correction** | Which skill handles a request | Update orchestrator routing table |
| **Template update** | Code scaffolding quality | Update skill reference file |
| **Default change** | Default behavior in a skill | Update SKILL.md default instruction |
| **New heuristic** | Detection rule for a class of problems | Add to routing-guide.md |
| **Anti-pattern block** | Prevent a recurring mistake | Add to relevant skill's anti-patterns |

## Learning Log Format

Track patterns in `memory/learning/patterns.md`:

```markdown
## Pattern: [Name]

**Type**: Failure | Win | Correction | Routing-miss
**Sessions observed**: [count]
**First seen**: [date]
**Last seen**: [date]

### Description
What happened: [what went wrong or right]
Trigger: [what caused it]
Impact: [what it cost — rework, corrections, broken builds]

### Root Cause
[why it keeps happening]

### Adaptation Applied
[what was changed and where — be specific]

### Verification
Next session: [did the adaptation help?]
Status: PENDING | RESOLVED | PARTIAL
```

## Session Learning Workflow

### At Session End
1. Review self-eval scores from the session — any dimension consistently < 3?
2. Count user corrections — what kept needing adjustment?
3. Note any loops that needed > 3 iterations — those are systemic issues
4. Add new entries to `memory/learning/patterns.md`
5. Check existing patterns — did any resolve? Did any worsen?

### At Session Start
1. Read `memory/learning/patterns.md` — what patterns are active?
2. Apply current adaptations (route differently, use updated templates)
3. Watch for recurrence of known patterns

## Behavior Adjustment Protocol

When a pattern reaches adaptation threshold (2+ sessions):

1. **Name it** — patterns need names to be tracked
2. **Locate the source** — which skill, template, or routing rule causes it?
3. **Write the adaptation** — specific, testable change
4. **Apply it** — update the relevant file (skill, reference, routing guide)
5. **Log it** — record in patterns.md with status PENDING verification

Do NOT adapt based on single events — wait for the second occurrence.

## What Good Learning Looks Like

After 10 sessions with active learning:
- Recurring user corrections drop by 50%+
- Loop iteration counts trend downward
- Routing accuracy improves (right skill, first try)
- New patterns are caught earlier (after 2 not 5 occurrences)

## Reference

Read `references/learning-patterns.md` for:
- Pattern detection templates with worked examples
- Adaptation decision trees
- Learning log file structure
- Inter-session pattern correlation methods
- When to escalate a pattern vs. adapt locally
