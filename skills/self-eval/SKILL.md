---
name: self-eval
description: "Self-evaluate task quality, track mistakes, improve over time. Use for evaluating work. Do NOT use for code review or testing."
---

# Self-Evaluation & Continuous Improvement

Every task is learning data. This skill embeds continuous self-evaluation into workflows, transforming sessions into compound learning. Instead of repeating mistakes, evaluate what worked, what failed, and refine the system.

## Why Self-Evaluation Matters

Without reflection, patterns blur. Self-evaluation enables:
- **Quality Improvement**: Spot what works vs. what fails
- **Mistake Prevention**: Understand root causes, avoid repetition
- **Workflow Optimization**: Identify fastest, most reliable approaches
- **Skill Refinement**: Surface gaps in the system itself

## Process: Post-Task Evaluation (5 minutes)

After each significant task:

1. **Summarize**: What was asked and delivered? Complexity level?
2. **Score Dimensions**: Rate 1-5 for Correctness, Completeness, Code Quality, Security, UX/DX
3. **Reflect**: What worked? What was hard? What would you change?
4. **Capture Learning**: Technique to remember, mistake to avoid, skill gap
5. **Overall**: Quality score 1-5 with specific improvement suggestions

## Quality Dimensions

- **Correctness (1-5)**: Handles all cases, edge cases, errors properly
- **Completeness (1-5)**: All requested features implemented and polished
- **Code Quality (1-5)**: Clean, maintainable, follows patterns
- **Security (1-5)**: Input validated, no vulnerabilities, safe defaults
- **UX/DX (1-5)**: Intuitive, fast, clear messages, feels good

## Quick Evaluation Template

```markdown
# Evaluation: [Task]

## Scores (1-5)
| Dimension | Score | Note |
|-----------|-------|------|
| Correctness | | |
| Completeness | | |
| Code Quality | | |
| Security | | |
| UX/DX | | |

## Reflection
- What worked: [decision/approach]
- What was hard: [challenge + lesson]
- Do differently: [specific change]

## Learning
- Pattern: [technique to remember]
- Mistake: [how to prevent]
- Skill gap: [improvement for system]
```

## Learning from Mistakes

Mistakes are data. Categorize them:

- **Logic Errors**: Off-by-one, wrong condition → Test all branches
- **Oversights**: Forgot validation → Checklist before shipping
- **Assumptions**: Wrong about behavior → Ask clarifying questions
- **Security**: Missed sanitization → Security checklist
- **Performance**: Inefficient code → Profile before optimizing

## Success Pattern Tracking

Document wins for reuse:

```markdown
# Pattern: [Name]

**Situation**: [When this applies]
**Approach**: [What worked]
**Result**: [Outcome]
**Use When**: [Similar scenarios]
```

## Session Retrospective

End of day/long session:

```markdown
# Retrospective: [Date]

## Wins
- [success 1]
- [success 2]

## Challenges & Learning
- [challenge + lesson learned]

## Patterns
- [observation about efficiency/approach]

## Improvements
- [gap in system]

## Next Session
- [priority]
```

## Reference Rubrics & Examples

Read `references/eval-criteria.md` for:
- Detailed 1-5 rubrics with concrete examples
- Security evaluation checklist
- UX/DX assessment guidance
- Scoring worksheet templates
- Common evaluation mistakes

---

**Next Steps**: After your next task, spend 5 minutes on evaluation. Score 1-5 for each dimension, capture one learning.
