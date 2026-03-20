# Memory Patterns Reference

Comprehensive guide for what to store, how to store it, and how memory flows through the system.

## What to Store vs. What to Derive

Memory exists to reduce redundant questions, not to replace thinking. Use this decision tree:

### STORE: Preferences & Patterns
- **Tech Stack Choices**: "React 18 + TypeScript + Jest + Vercel" (stable across projects)
- **Coding Conventions**: "Always use named exports", "JSDoc with @param/@returns"
- **Architecture Patterns**: "Component composition pattern", "custom hooks for logic"
- **Team Practices**: "Code review expectations", "deployment workflow"
- **Past Decisions & Rationale**: "Why we chose PostgreSQL over MongoDB"
- **Common Templates**: Error handlers, component boilerplate, API patterns

### DERIVE (Don't Store): Runtime Data
- Specific commit hashes or branch names
- API response formats (look up live)
- Library versions (check package.json)
- Test results or debug logs
- Temporary state or context
- Code that changes monthly

### Example Distinction
- STORE: "I prefer functional components with hooks"
- DON'T STORE: "CurrentUser component uses useState at line 45"
- STORE: "Error handling: try-catch with typed error classes"
- DON'T STORE: "TypeError at src/api.ts:123"

## Preference Detection Heuristics

Claude auto-detects and proposes memory updates based on coding patterns.

### Stack Detection (Tool/Framework Frequency)

**Rule: 3+ occurrences in same session = offer to remember**

```
User writes:
  - app.tsx uses React.FC
  - Button.tsx uses React.FC
  - Modal.tsx uses React.FC

Proposal: "I see you're consistently using React.FC. Should I remember React 18 as your default?"
```

**Tool Detection Examples:**
- Tailwind in 3+ files → "CSS: TailwindCSS with [your config]"
- Redux in multiple components → "State Management: Redux with middleware"
- GraphQL queries in API folder → "API: GraphQL (not REST)"
- Docker config found → "Deployment: Docker containers"

### Style Detection (Naming & Organization Patterns)

**Rule: Same pattern used 2+ times consistently = worth remembering**

```
Detected patterns:
  - useCustomHook.ts, useAnotherHook.ts, useThirdHook.ts
  - Proposal: "Storing custom hook pattern: use[PascalCase].ts in hooks/"

  - src/Button/Button.tsx, src/Modal/Modal.tsx
  - Proposal: "Storing component structure: Component/Component.tsx"

  - error.status, error.message, error.code (consistent error objects)
  - Proposal: "Error object shape: { status, message, code }"
```

### Import/Export Patterns

```
Detected: export const Foo = ... (not export default)
Used in 3+ files

Proposal: "I notice you prefer named exports. Remember this convention?"
Stored as: "Exports: Named (not default)"
```

### Comment & Documentation Style

```
Detected:
  /** @param userId User's unique identifier */
  function getUser(userId: string) { ... }

Proposal: "JSDoc style detected. Should I remember JSDoc with @param/@returns?"
```

## Context Window Management

Not all memory belongs in CLAUDE.md hot cache. Use promotion/demotion based on usage frequency.

### Hot Cache Criteria (CLAUDE.md)
- Used in every session or nearly every session
- Critical to current sprint/project
- Reference point for 80%+ of decisions
- Fits in 150-word summary
- Examples:
  - Primary language and framework
  - Current active project(s)
  - Core coding style rules
  - Team conventions affecting daily work

### Deep Storage Criteria (memory/ subdirectories)
- Used 1-2 times per week
- Detailed reference material
- Project-specific context
- Legacy or backup information
- Examples:
  - Specific library config documentation
  - Architecture decision rationale
  - Old project context
  - Team member contact info
  - Archived patterns

### Archive Criteria (memory/archive/)
- Unused for 6+ months
- Superseded by newer approaches
- Historical context only
- Examples:
  - Previous tech stacks you've moved away from
  - Old project documentation
  - Deprecated patterns
  - Team members who've left

**Promotion Flow Example:**
```
Week 1: Stored in memory/patterns/form-validation.md
Week 2: Used 3 times
Week 3: Used 2 times
→ Proposal: "Form validation is core to your work. Move to CLAUDE.md?"

Demotion Flow Example:
Last used: 3 months ago
Status: "This pattern is rarely used now"
→ Proposal: "Archive old-auth-pattern.md to memory/archive/?"
```

## Memory Conflict Resolution

When old and new information conflict, apply these rules:

### Scenario 1: Tech Stack Change
```
CLAUDE.md says: "Frontend: React 18"
User writes: "Let's rebuild in Vue 3"

Resolution:
- Ask: "Are you switching Vue 3 for this project, or as your new default?"
- If project: Store in local CLAUDE.md override
- If personal: Update global CLAUDE.md, archive old React docs
```

### Scenario 2: Style Evolution
```
CLAUDE.md says: "Prefer class components"
User writes: 5 functional components with hooks

Resolution:
- Observe: Clear pattern shift
- Proposal: "Your style has evolved to functional components. Update CLAUDE.md?"
- Update confirmed: Replace in CLAUDE.md, archive old pattern
```

### Scenario 3: Conflicting Patterns
```
memory/patterns/ has two error handling approaches
User clearly uses one approach 80% of the time

Resolution:
- Flag redundancy: "Two error patterns stored. One is used much more often."
- Proposal: Archive less-used pattern, promote dominant one
```

**Conflict Resolution Rules:**
1. Most recent usage wins (with reasonable time window)
2. Explicit user statements override inferred patterns
3. Project-local CLAUDE.md overrides user defaults
4. When unsure, ask user for clarification
5. Archive old info instead of deleting (recovery possible)

## Memory Format Specifications

### CLAUDE.md Schema

```markdown
# CLAUDE.md

## Me
- Role: [single-line role description]
- Experience Level: [Junior/Mid/Senior/Staff]
- Timezone: [TZ name]
- Current Focus: [current work]

## Stack Preferences
- Languages: [comma-separated]
- Frameworks: [comma-separated with versions]
- Build Tools: [specific tools used]
- Package Manager: [npm/yarn/pnpm]
- Testing: [primary testing framework]
- Deployment: [primary deployment target]

## Coding Style
- Formatting: [formatter name + key rules]
- Naming: [camelCase/PascalCase/snake_case usage]
- Comments: [JSDoc/docstring approach]
- Error Handling: [try-catch, custom errors, patterns]
- Performance: [key optimization strategies]

## Active Projects
- **project-name**: [1-line description + stack]

## Terms & Shortcuts
- "ACRONYM": meaning
- "abbrev": meaning

## Deep Memory
See memory/ for deeper documentation
```

**Size Limits:**
- Total CLAUDE.md: < 300 words
- Each section: 1-2 sentences
- Stack section: 3 items max per category
- Active projects: Current 2-3 only

### memory/ Directory Schema

**Stack docs** (memory/stack/):
```markdown
# [Framework/Tool Name]

## Version & Setup
- Version: X.Y.Z
- Installation: [how you install it]
- Key Config: [your specific configuration]

## Key Practices
- [Practice 1]
- [Practice 2]
```

**Project docs** (memory/projects/):
```markdown
# [Project Name]

## Context
- Purpose: [what it does]
- Tech Stack: [full stack]
- Team: [who works on it]
- Key Files: [critical files]

## Architecture
[brief overview]

## Current Status
[active work, sprint, blockers]
```

**Decision docs** (memory/decisions/):
Use ADR (Architecture Decision Record) format:
```markdown
# ADR-001: [Title]

## Status
Accepted/Superseded

## Context
[Why we faced this decision]

## Decision
[What we decided and why]

## Consequences
[Positive and negative outcomes]
```

**Pattern docs** (memory/patterns/):
```markdown
# [Pattern Name]

## Problem
[What situation does this solve?]

## Solution
[The pattern itself]

## Code Example
[Real example from your codebase]

## When to Use
[Guidance on applying this pattern]

## Alternatives
[Other approaches considered]
```

## Examples of Effective Memory Entries

### Good: React Custom Hooks Pattern
```markdown
# Custom Hooks Pattern

## Problem
Logic duplication across components

## Solution
Extract logic into useX hooks

## Example
```typescript
// hooks/useFormValidation.ts
export function useFormValidation(fields: FieldSchema[]) {
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  return { errors, touched, validate, reset };
}
```

## When to Use
- Complex form logic
- Reusable component logic
- Side effect management
```

### Good: API Error Handling Strategy
```markdown
# API Error Handling

## Pattern
All API errors extend ApiError with:
- status: HTTP status code
- code: Application error code
- message: User-friendly message
- details: Developer debugging info

## Implementation
```typescript
class ApiError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string,
    public details?: unknown
  ) {
    super(message);
  }
}
```

## Usage
```typescript
try {
  const data = await fetch(...);
} catch (e) {
  throw new ApiError(500, 'FETCH_FAILED', 'Failed to load data');
}
```
```

### Bad: Memory Entry (Too Specific)
```markdown
# Bug Fix from Session March 19

Fixed issue where Button component wasn't re-rendering...
Details: Line 45 of Button.tsx...
```
Problem: Specific to one bug, not a pattern. Store if it reveals a general issue, not the bug itself.

### Bad: Memory Entry (Outdated)
```markdown
# Setting up Jest

npm install --save-dev jest
Then run: jest --init
...
```
Problem: Public documentation, doesn't reflect your specific setup. Link to docs instead.

## Tracking Memory Over Time

### Metrics to Monitor
- **Hot Cache Hit Rate**: How often is CLAUDE.md sufficient? Target: 85%+
- **Deep Storage Usage**: How often do you reference memory/ docs? Target: 60%+
- **Update Frequency**: How often does memory shift? Target: Monthly
- **Archive Growth**: How much becomes obsolete? Target: < 20% archived

### Improvement Signals
- More than 2 clarifying questions per session → improve CLAUDE.md clarity
- Repeatedly re-explaining same concept → promote to hot cache
- Memory section never accessed → demote or archive
- Confused by conflicting entries → clean up redundancy

---

**Next**: Configure your memory/ structure and start with CLAUDE.md. Review quarterly for archival and promotion.
