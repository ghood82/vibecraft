---
name: loop
description: "Iterative run→check→fix→repeat cycles. Use for test-fix loops, build-fix cycles, quality gate retries, retry-until-passing, lint loops, green-CI convergence. Do NOT use for one-shot test runs (use code-quality) or pipeline setup (use cicd)."
---

# Loop — Iterative Convergence

Run something, check the result, fix what failed, repeat until it passes. This skill manages iterative loops so you don't spiral into untracked retry hell.

## When to Activate

- "Keep running tests until they pass"
- "Fix all lint errors"
- "Retry until the build is green"
- "Run the quality gate, fix issues, repeat"
- Any test-fix-rerun cycle

## Core Loop Protocol

```
LOOP(max=N, exit=condition):
  iteration = 1
  while iteration <= max:
    1. RUN  — execute the command
    2. CHECK — did it pass? → EXIT (success)
    3. ANALYZE — what failed and why?
    4. FIX — apply targeted fix
    5. iteration++
  → EXIT (max iterations reached, report remaining failures)
```

**Default max iterations**: 5. Always set this — infinite loops are bugs.

## Loop Types

### Test-Fix Loop
```
Command: npm test / pytest / vitest run
Exit: all tests pass (exit code 0)
Fix strategy: address one failing test group at a time, not all at once
Max: 5 iterations
```

### Lint Loop
```
Command: npm run lint / eslint . / biome check
Exit: zero errors (warnings allowed)
Fix strategy: auto-fix first (--fix), then manual for remaining
Max: 3 iterations (if still failing after 3, report and stop)
```

### Build-Fix Loop
```
Command: npm run build / tsc --noEmit / cargo build
Exit: exit code 0, no errors
Fix strategy: fix type errors top-down (errors cascade — fix root causes first)
Max: 4 iterations
```

### Quality Gate Loop
```
Command: full pipeline (lint → test → build)
Exit: all stages green
Fix strategy: lint first (fastest), then tests, then build
Max: 6 iterations (pipeline is expensive — set higher max)
```

## Loop Progress Tracking

Report after each iteration:
```
Iteration 2/5: 3 tests failing (down from 7)
  Fixed: auth.test.js ✓, user.test.js ✓
  Remaining: payment.test.js (mock not resolving)
```

Stop early if:
- No progress between iterations (same failures) → diagnose root cause instead
- New failures introduced by fixes → revert last fix, try different approach
- Max iterations reached → surface remaining failures, stop

## Anti-Patterns to Avoid

- **Brute-force retrying**: Same fix repeated → stop, think differently
- **Fixing symptoms**: Test mocks wrong → fix the actual logic, not the mock
- **Cascading guesses**: Each fix breaks something new → revert to last-known-good
- **Ignoring the exit condition**: Loop must have a clear "done" state

## Reference

Read `references/loop-patterns.md` for:
- Detailed iteration templates for each loop type
- Convergence heuristics (when to stop vs. push through)
- Parallel fix strategies for independent failures
- Integration with CI/CD green-gate patterns
