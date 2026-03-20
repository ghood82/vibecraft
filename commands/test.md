---
description: Generate and run tests
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
argument-hint: [file or feature to test]
---

Generate tests and/or run the test suite.

Read the testing reference at `${CLAUDE_PLUGIN_ROOT}/skills/code-quality/references/testing-strategies.md`.

**If $ARGUMENTS specifies a file or feature:**
1. Read the source code to understand what needs testing
2. Identify the testing framework in use (Vitest, Jest, Playwright, pytest)
3. Generate tests covering: happy path, edge cases, error cases
4. Write test files following project conventions
5. Run the tests and report results

**If no arguments (run all tests):**
1. Detect test command from package.json scripts
2. Run the test suite
3. Report results with pass/fail summary
4. If failures: diagnose and offer fixes

**Test quality standards:**
- Test behavior, not implementation
- Descriptive test names
- One concept per test
- Cover happy path + 2-3 edge cases minimum
