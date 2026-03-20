# Loop Patterns — Reference

Detailed templates, heuristics, and anti-patterns for iterative fix cycles.

## Iteration Templates

### Standard Test-Fix Iteration

```
## Iteration [N]/[MAX] — Test-Fix Loop

### Run
Command: `npm test -- --run` / `python -m pytest`
Result: [PASS ✓ | FAIL ✗]

### Failures
| Test | Error | Category |
|------|-------|----------|
| [test name] | [error message] | [type error / missing mock / assertion / async] |

### Root Cause Analysis
- Primary cause: [what's actually broken]
- Secondary: [cascading failures from primary]

### Fix Plan
1. [specific fix for root cause]
2. [fix for secondary if independent]

### Applied Fix
Files changed: [list]
Strategy: [what was changed and why]

### Next Iteration
Expected outcome: [which tests should now pass]
```

### Lint Loop Template

```
## Lint Iteration [N]/[MAX]

### Phase 1: Auto-Fix
Command: `eslint . --fix` / `biome check --apply`
Fixed automatically: [count] issues

### Phase 2: Manual Review
Remaining issues:
| File | Line | Rule | Fix |
|------|------|------|-----|
| [file] | [L#] | [rule] | [fix description] |

### Applied Fixes
[files and specific changes]

### Status
Errors remaining: [count]
Continue: [YES if errors remain] / [DONE if zero errors]
```

### Build / Type-Fix Template

```
## Build Iteration [N]/[MAX]

### Errors (root causes only — cascading errors omitted)
| File | Error | Type |
|------|-------|------|
| [file] | [TS error or build error] | [missing type / wrong arg / import] |

### Fix Strategy
Fix root cause files first. Cascading errors in dependent files often auto-resolve.

### Changes
[specific type fixes applied]

### Verification
Run: `tsc --noEmit` (type check only, fast)
Then: `npm run build` (full build)
```

## Convergence Heuristics

### Keep Going When
- Failure count is decreasing each iteration
- Fixes are targeted (one failure = one fix, not shotgun changes)
- New passes outnumber new failures

### Stop and Diagnose When
- Same failures appear in 2 consecutive iterations → you're fixing the wrong thing
- Failure count is increasing → fixes are introducing regressions
- You're on iteration 3+ with no progress → architectural issue, not a fix-loop problem

### Hard Stop Conditions
- **Max iterations reached**: Surface remaining failures. Do NOT continue silently.
- **Circular dependency**: Fix A breaks B, fix B breaks A → escalate to design decision
- **External dependency**: Failure caused by external service/API → can't fix in loop, report and exit

## Parallel Fix Strategy

When failures are independent, fix in parallel batches:

```
Iteration 3 — 4 failures, all independent:
  Batch A (auth tests): missing mock for getUserById
  Batch B (payment tests): wrong assertion on amount format
  → Fix both in one iteration, re-run, expect all 4 to pass
```

When failures are dependent, fix in sequence:
```
TypeError in utils → cascades to 6 test failures
  Fix utils first → all 6 should resolve
  Don't fix individual test assertions until root cause is clear
```

## Integration with CI/CD

Use loop patterns as local pre-flight before pushing:

```
Pre-push loop:
1. npm run lint --fix (auto-fix lint)
2. npm test -- --run (test loop, max 3 iterations)
3. npm run build (build check, max 2 iterations)
4. git push (only if all green)
```

If local loop can't converge → do NOT push → file a todo, investigate deeper.

## Quality Gate Pattern

Full pipeline loop for production-readiness:

```
quality-gate-loop (max 6 iterations):
  Stage 1: Lint (fastest — fix these first)
  Stage 2: Type check (tsc --noEmit — fast feedback)
  Stage 3: Unit tests
  Stage 4: Integration tests (if present)
  Stage 5: Build
  → All green = ready to ship
```

Run stages in order — don't run integration tests until unit tests pass.

## Reporting Format

After loop completes, always output:

```
## Loop Complete — [PASS ✓ | MAX REACHED ✗]

Iterations used: [n]/[max]
Final state: [all passing | X failures remain]

### Passed
- [test/check that now passes]

### Remaining Failures (if any)
| Issue | Root Cause | Recommended Next Step |
|-------|-----------|----------------------|
| [failure] | [cause] | [action] |

### Changes Made
[summary of all fixes applied across iterations]
```
