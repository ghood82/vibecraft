# Changelog

All notable changes to VibeCraft are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.8.0] — 2026-03-19

### Added
- **Multi-agent orchestration**: Agent Dispatch section in orchestrator with routing table, skill-vs-agent disambiguation, parallel and sequential pipeline patterns
- **agent-workflows** skill: Pre-built multi-agent workflow pipelines (PR Review, Ship-Ready Check, Full Audit, New Feature Pipeline, Security Remediation, Accessibility Overhaul)
- Cross-agent awareness: All 5 agents now include sibling agent knowledge and Handoff Recommendations in their output templates
- Agent output chaining: Structured handoff protocol enabling agent → agent pipelines
- "multi-agent", "subagents", "agent-workflows" keywords in plugin.json

### Changed
- Updated orchestrator SKILL.md with Agent Dispatch section separate from skill routing table
- Updated all 5 agent definitions (code-reviewer, security-scanner, researcher, qa-tester, ux-auditor) with cross-agent handoff recommendations
- Updated SessionStart hook banner to show 34 skills and v0.8.0
- Bumped version to 0.8.0
## [0.7.0] — 2025-03-19

### Added
- Full documentation suite: README (rewritten), GETTING-STARTED.md, CONTRIBUTING.md, SECURITY.md, CHANGELOG.md, LICENSE, skills/INDEX.md
- Skill catalog (skills/INDEX.md) with all 33 skills organized by category
- "documentation" keyword in plugin.json

### Changed
- Renamed plugin from "Ultimate Vibe Coder" to **VibeCraft** across all files
- Rewrote README.md with complete skill table (33 skills, up from 21 listed), accurate hook count (5), and professional formatting
- Updated SessionStart hook banner to display VibeCraft branding
- Bumped version to 0.7.0

## [0.6.0] — 2025-03-10

### Added
- **secrets-management** skill: Enterprise secrets with Vault, AWS/GCP/Azure secret managers, SOPS, sealed secrets, rotation policies
- **compliance** skill: HIPAA, SOC 2, GDPR, PCI DSS code-level enforcement, audit logging, data privacy, compliance-as-code

### Changed
- Renamed project internally to VibeCraft (partial — completed in v0.7.0)

## [0.5.0] — 2025-02-28

### Added
- **project-scaffolding** skill: Bootstrap new projects from scratch with best-practice structure, config, and boilerplate for any stack
- **monorepo-management** skill: Turborepo, Nx, pnpm workspaces, package boundaries, selective builds, shared libraries
## [0.4.0] — 2025-02-15

### Added
- **semantic-memory** skill: Vector-based semantic search over past sessions and project knowledge
- **webhooks** skill: Event-driven triggers — respond to GitHub events, API calls, Slack messages
- **daemon-mode** skill: Persistent background agent for 24/7 monitoring and auto-response
- **context-engine** skill: Pluggable context management with token budgets and smart windowing
- **multi-platform** skill: Integration with Slack, Discord, Telegram, WhatsApp, and email

## [0.3.0] — 2025-02-01

### Added
- **loop** skill: Iterative run→check→fix→repeat cycles for test-fix loops, build-fix cycles, and quality gate retries
- **scheduler** skill: Cron jobs, recurring tasks, scheduled automation, and dependency update schedules
- **structured-logging** skill: Structured log output with timestamps, log levels, session log files, and JSON formatting
- **learning** skill: Closed-loop self-improvement — track failures and successes, detect recurring patterns, adjust behavior

## [0.2.0] — 2025-01-18

### Added
- **orchestrator** skill: Central brain with intent detection, MCP-aware routing, and workflow management
- 5 specialized agents: code-reviewer, security-scanner, researcher, qa-tester, ux-auditor
- 10 slash commands: /vibe, /ship, /review, /research, /plan, /secure, /test, /improve, /status, /feature
- Hooks system with SessionStart banner, PreToolUse safety gates, and PostToolUse monitors

## [0.1.0] — 2025-01-05

### Added
- Initial release with core vibe-coding skills
- Skills: vibe-coding, code-quality, deployment, research, ui-ux, security, memory, self-eval, docs-productivity, ai-agents, git-workflow, api-design, performance, monitoring, database, env-secrets, linting, cicd, codebase-understanding
- Basic project scaffolding and code generation
- Multi-platform deployment support
- OWASP security scanning
- Persistent two-tier memory system