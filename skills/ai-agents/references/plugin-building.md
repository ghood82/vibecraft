# Claude Code Plugin Building Guide

## Plugin Structure
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifest (required)
├── commands/                 # Slash commands
│   └── my-command.md
├── skills/                   # Auto-triggered knowledge
│   └── my-skill/
│       ├── SKILL.md
│       └── references/
├── agents/                   # Autonomous subagents
│   └── my-agent.md
├── hooks/                    # Event-driven automation
│   └── hooks.json
├── .mcp.json                 # MCP server config
└── README.md
```

## Skills: Domain Knowledge

Skills trigger automatically when Claude detects a relevant request.

**SKILL.md Structure:**
```yaml
---
name: my-skill
description: >
  This skill should be used when the user asks to "do X",
  "create Y", or needs guidance on Z. Also triggers for
  "specific phrase 1", "specific phrase 2".
---
```

**Best practices:**
- Keep SKILL.md under 3000 words — move depth to references/
- Description is the trigger mechanism — be pushy with trigger phrases
- Use imperative voice ("Parse the config" not "You should parse")
- Explain WHY behind instructions, not just WHAT
- Include output format templates

## Commands: User-Initiated Actions

```markdown
---
description: Short description for /help (under 60 chars)
allowed-tools: Read, Write, Edit, Bash(git:*)
argument-hint: [file-path]
---

Instructions for Claude. Use $ARGUMENTS for user input.
Use @$1 to include file contents.
Use !`command` for inline bash execution.
```

## Agents: Autonomous Workers

```markdown
---
name: my-agent
description: When to spawn this agent.

<example>
Context: ...
user: "..."
assistant: "I'll use the my-agent agent."
<commentary>Why this agent is appropriate.</commentary>
</example>

model: inherit
color: blue
tools: ["Read", "Grep", "Glob"]
---

System prompt for the agent. Define:
- Core responsibilities
- Analysis process
- Output format
```

## Hooks: Event Automation

```json
{
  "PreToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [{
      "type": "prompt",
      "prompt": "Check this file write for issues.",
      "timeout": 15
    }]
  }],
  "SessionStart": [{
    "matcher": "",
    "hooks": [{
      "type": "command",
      "command": "cat ${CLAUDE_PLUGIN_ROOT}/context/project.md",
      "timeout": 5
    }]
  }]
}
```

Events: PreToolUse, PostToolUse, Stop, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit

## Packaging

```bash
cd my-plugin
zip -r /tmp/my-plugin.plugin . -x "*.DS_Store"
```

The .plugin file can be installed by opening it or dragging into Claude Code.

## Skill Writing Tips

1. **Progressive disclosure**: SKILL.md → references/ → scripts/
2. **Clear triggers**: Include 8-12 trigger phrases in the description
3. **Structured outputs**: Provide templates for every output type
4. **Severity ratings**: Classify findings/recommendations by impact
5. **Decision trees**: Use conditional logic for different contexts
6. **Memory integration**: Note what to remember after each use
7. **Fallbacks**: Define what to do when things go wrong
