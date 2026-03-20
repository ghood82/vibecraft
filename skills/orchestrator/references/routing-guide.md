# Routing Decision Guide

Detailed decision trees for routing complex or ambiguous requests to the right sub-skills.

## Multi-Signal Routing

When a request matches multiple skills, use this precedence:

### Security Overrides
If ANY of these are present, route through security FIRST:
- Authentication/authorization code
- API keys, secrets, tokens
- User data handling (PII, passwords)
- Payment processing
- File uploads
- Database queries with user input
- Third-party integrations

Then continue to the originally intended skill.

### Stack Detection
Auto-detect the project stack from config files:

| File Present | Stack Detected | Default Tooling |
|-------------|---------------|-----------------|
| `next.config.*` | Next.js | Vercel deployment, React components, App Router |
| `wrangler.toml` | Cloudflare Workers | Workers deployment, D1/R2/KV |
| `package.json` with `"type": "module"` | Modern Node.js | ESM imports, native fetch |
| `pyproject.toml` / `requirements.txt` | Python | pip, venv, FastAPI/Django |
| `Cargo.toml` | Rust | cargo, compile-first workflow |
| `go.mod` | Go | go build, standard library preference |
| `docker-compose.yml` | Containerized | Docker deployment, multi-service |
| `turbo.json` / `pnpm-workspace.yaml` | Monorepo | Turborepo/pnpm workspace commands |

### Ambiguous Requests

**"Make it better"**
→ Run code-quality review first to identify what "better" means
→ Then route to the specific improvement area

**"Help me with this"** (with file context)
→ Analyze the file type and current state
→ Route based on what's most needed (tests? refactor? docs?)

**"I'm stuck"**
→ Check git status for current work in progress
→ Read recent files for context
→ Diagnose the blocker, then route to the right skill

**"Let's start a new project"**
→ Ask ONE question: "What are you building?"
→ Route to vibe-coding for scaffolding
→ Auto-detect preferred stack from memory or ask

## Complexity Estimation

Before executing, estimate complexity to decide on approach:

| Complexity | Time Estimate | Approach |
|-----------|--------------|----------|
| **Trivial** | < 2 minutes | Just do it inline, no ceremony |
| **Simple** | 2-10 minutes | Single skill, minimal coordination |
| **Medium** | 10-30 minutes | Multi-skill pipeline, use TodoList |
| **Complex** | 30+ minutes | Full orchestration, parallel agents, checkpoints |

## Error Recovery

When something goes wrong mid-workflow:

1. **Build error** → Read the error, fix inline if obvious, route to code-quality if complex
2. **Test failure** → Show the failure, suggest fix, re-run
3. **Deploy failure** → Check logs, fix config, retry. Common issues: env vars, build command, node version
4. **Security finding** → Block deployment until resolved, show severity and fix
5. **Research dead-end** → Broaden search terms, try alternative sources, fall back to knowledge

## Connected MCP Integrations

The user has these MCP tools available. Leverage them when relevant:

| MCP | Capabilities | When to Use |
|-----|-------------|-------------|
| **Vercel** | Deploy, list projects, get logs, check domains | Deployment, preview URLs, build debugging |
| **Cloudflare** (x2) | Workers, D1, R2, KV, Pages | Edge deployment, database, storage |
| **Atlassian (Jira/Confluence)** (x2) | Create/edit issues, search, create pages | Task tracking, documentation, project management |
| **Figma** | Get design context, screenshots, variables, code connect | Design-to-code, design tokens, visual reference |
| **Google Drive** | Search, fetch files | Access existing docs, spreadsheets, resources |
| **Context7** | Library docs lookup | Get up-to-date docs for any library/framework |
| **Things** | Task management (todos, projects, inbox) | Personal task tracking, GTD workflow |
| **AWS** | Call AWS APIs | Infrastructure management, S3, Lambda |
| **ElevenLabs** | TTS, sound effects, music, voice agents | Audio generation, voice features |
| **Chrome Control** | Navigate, execute JS, read pages | Browser testing, scraping, automation |
| **Desktop Commander** | File system, processes, search | System-level file operations |
| **PowerPoint MCP** | Create/edit presentations | Slide decks, pitch decks |
| **Word MCP** | Create/edit Word docs | Document generation |
| **PDF Tools** | Fill, analyze, extract PDFs | PDF manipulation |

### MCP-Aware Routing

When the user's request involves these platforms, prefer MCP tools over manual approaches:

- **"Deploy to Vercel"** → Use Vercel MCP `deploy_to_vercel`, not manual CLI
- **"Create a Jira ticket"** → Use Atlassian MCP `createJiraIssue`
- **"Check the Figma design"** → Use Figma MCP `get_design_context` + `get_screenshot`
- **"What does the library docs say about X"** → Use Context7 MCP `resolve-library-id` then `get-library-docs`
- **"Add this to my tasks"** → Use Things MCP `add_todo`
- **"Set up a D1 database"** → Use Cloudflare MCP `d1_database_create`
- **"Store files in R2"** → Use Cloudflare MCP `r2_bucket_create`

### Cross-MCP Workflows

Complex tasks may chain multiple MCPs:
```
"Build feature from Figma design, create Jira ticket, deploy to Vercel"
→ Figma MCP: get_design_context → extract design tokens and structure
→ vibe-coding skill: generate code from design
→ Atlassian MCP: createJiraIssue → track the work
→ git-workflow: commit and push
→ Vercel MCP: deploy_to_vercel → ship it
```

## Agent Spawning Guidelines

Spawn specialized agents when:
- The task benefits from focused attention (deep code review)
- Multiple independent tasks can run in parallel
- The task requires a different model capability (e.g., haiku for fast linting)

Keep work inline when:
- The task is simple and quick
- Context from the current conversation is crucial
- The user is actively collaborating and wants to see progress in real-time
