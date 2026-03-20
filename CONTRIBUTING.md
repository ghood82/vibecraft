# Contributing to VibeCraft

VibeCraft is designed to be extensible. This guide covers how to add new skills, hooks, agents, and commands, plus the style conventions that keep everything consistent.

## Plugin Structure

```
vibecraft/
├── .claude-plugin/plugin.json    # Manifest — name, version, keywords
├── skills/                       # Each skill is a directory
│   └── my-skill/
│       ├── SKILL.md              # Skill definition (required)
│       └── references/           # Optional reference docs
│           └── patterns.md
├── agents/                       # Agent definitions (markdown files)
├── commands/                     # Slash command definitions (markdown files)
├── hooks/hooks.json              # Lifecycle hooks
└── README.md
```

## Adding a New Skill

### 1. Create the Directory

```bash
mkdir -p skills/my-new-skill/references
```

### 2. Write SKILL.md

Every skill needs a `SKILL.md` file with YAML frontmatter and a body. Here's the template:
```yaml
---
name: my-new-skill
description: "One-line description of what this skill does. Use for trigger1, trigger2, trigger3. Do NOT use for anti-trigger1 (use other-skill instead)."
---
```

The `description` field is critical — it controls when the orchestrator activates your skill. Follow this pattern:

- Start with a brief description of the skill's purpose.
- Add `Use for` followed by specific trigger phrases and keywords.
- Add `Do NOT use for` to prevent false activations, pointing to the correct skill instead.

### 3. Write the Body

The body of SKILL.md contains the instructions Claude follows when the skill is activated. Structure it with progressive disclosure:

```markdown
# My New Skill

## WHEN to Use
- Trigger condition 1
- Trigger condition 2

## WHEN NOT to Use
- Anti-pattern 1 → use `other-skill` instead
- Anti-pattern 2 → use `another-skill` instead

## Core Workflow
1. Step one
2. Step two
3. Step three

## Reference Files
- `references/patterns.md` — detailed patterns and examples
```
### 4. Add Reference Files (Optional)

Reference files go in `skills/my-new-skill/references/` and contain detailed patterns, templates, checklists, or examples that the skill can pull in. Keep each reference file focused on one topic.

### 5. Update the Index

Add your skill to `skills/INDEX.md` in the appropriate category.

## Adding a Hook

Hooks are defined in `hooks/hooks.json`. There are three lifecycle events:

| Event | When It Fires | Use For |
|-------|--------------|---------|
| `SessionStart` | Once when a session begins | Banners, initialization, context loading |
| `PreToolUse` | Before a tool executes | Validation, safety checks, blocking |
| `PostToolUse` | After a tool executes | Monitoring, logging, quality checks |

Each hook has a `matcher` (which tool names to match, using `|` for OR) and a `hooks` array. Two hook types are available:

- **`command`**: Runs a shell command. Return non-empty text to display a message (or `BLOCK:` prefix to block execution).
- **`prompt`**: Sends a prompt to Claude for evaluation. Used for nuanced checks that regex can't handle.

Example — add a hook that warns when editing test files:

```json
{
  "matcher": "Write|Edit",
  "hooks": [
    {
      "type": "command",
      "command": "echo $TOOL_INPUT | grep -q '__tests__\\|.test.\\|.spec.' && echo 'NOTE: Modifying test files — ensure tests still pass.' || echo ''",
      "timeout": 1
    }
  ]
}
```
## Adding an Agent

Agents are markdown files in `agents/`. Each agent definition specifies the agent's role, personality, and specialty. The existing agents follow a pattern of color-coded identities and focused expertise areas.

## Adding a Command

Commands are markdown files in `commands/`. The filename becomes the slash command (e.g., `review.md` → `/review`). Each command file contains the instructions Claude follows when the command is invoked.

## Style Guide

Follow these conventions to keep VibeCraft consistent:

### Skill Descriptions
- Keep the `description` frontmatter field to one line.
- Always include `Use for` triggers and `Do NOT use for` anti-triggers.
- Use imperative mood: "Scan for vulnerabilities" not "Scans for vulnerabilities."

### Reference Files
- One topic per file. Don't combine unrelated patterns.
- Use descriptive filenames: `supabase-patterns.md`, not `ref1.md`.
- Keep individual reference files under 500 lines. Split if needed.

### Progressive Disclosure
- SKILL.md body should give Claude enough to handle 80% of cases.
- Reference files handle the remaining 20% — deep patterns, edge cases, templates.
- Don't duplicate content between SKILL.md and references.

### Line Limits
Following Anthropic's best practices for plugin skills:
- SKILL.md frontmatter: keep description under 200 characters.
- SKILL.md body: aim for 50–150 lines. Anything longer should be split into references.
- Reference files: aim for 100–300 lines each.

## Testing Changes

1. Re-zip the plugin directory into a `.plugin` file.
2. Install it in Claude Code: `claude plugin add vibecraft.plugin`
3. Start a new session and verify the SessionStart banner shows correctly.
4. Test your changes by triggering the relevant skill, command, or hook.
5. Check that the orchestrator routes to your skill for the intended triggers and does NOT route to it for anti-triggers.