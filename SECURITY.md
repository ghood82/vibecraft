# Security Policy

VibeCraft takes security seriously. This document describes the security mechanisms built into the plugin, their limitations, the compliance skill's coverage, and how to report vulnerabilities.

## Security Hooks

VibeCraft ships with three hook-based security layers that run automatically on every session. These are defined in `hooks/hooks.json` and require no configuration.

### 1. Secret Scanner (PreToolUse — Write/Edit)

**What it does:** Before any file write or edit is committed, a prompt-based hook asks Claude to scan the content for hardcoded API keys, passwords, tokens, database connection strings, and other secrets. If a real secret is detected, the write is blocked.

**What it catches:**
- Hardcoded API keys (e.g., `sk-live-...`, `AKIA...`)
- Plaintext passwords and tokens
- Database connection strings with credentials
- Bearer tokens and auth headers in code

**What it does NOT catch:**
- Secrets that are obfuscated or split across multiple variables
- Secrets written to files via Bash commands (bypasses Write/Edit matcher)
- Secrets in environment variable files that are intentionally secret stores
### 2. Dangerous Command Blocker (PreToolUse — Bash)

**What it does:** Before any Bash command runs, a regex-based hook checks for dangerous patterns and blocks execution if a match is found.

**What it catches:**
- Secret exfiltration via curl/wget with auth headers
- Piping base64-encoded payloads to bash
- Recursive force-delete of root paths (`rm -rf /`)
- Force-pushing to main/master branches

**What it does NOT catch:**
- Novel exfiltration techniques not matching the regex
- Destructive commands that don't match the specific patterns
- Commands that are dangerous in context but benign in isolation

### 3. Prompt Injection Defense (PostToolUse — WebFetch/WebSearch)

**What it does:** After any web content is fetched, a regex-based hook scans the output for common prompt injection patterns and warns if detected.

**What it catches:**
- "Ignore previous instructions" patterns
- "You are now" role-hijacking attempts
- "Reveal your system prompt" extraction attempts
- Fake `<system>` tags and sudo/root mode attacks

**What it does NOT catch:**
- Sophisticated or novel injection techniques
- Encoded or obfuscated injection payloads
- Injections embedded in legitimate-looking content
### Additional Monitors

Two additional PostToolUse hooks provide quality and reliability feedback:

- **Agent Quality Gate** (PostToolUse — Agent): When a sub-agent completes, logs the outcome and suggests re-running with a refined prompt if quality is low.
- **Bash Failure Detector** (PostToolUse — Bash): Detects test/build failures in command output and suggests engaging the loop engine for iterative fix cycles.

## What the Hooks Do NOT Protect Against

The security hooks are a first line of defense, not a comprehensive security solution. They do not protect against:

- Secrets written via Bash commands (only Write/Edit is scanned)
- Sophisticated or zero-day prompt injection techniques
- Social engineering or context manipulation attacks
- Supply chain attacks in dependencies
- Secrets that are already committed to version control
- Runtime vulnerabilities in generated code

Always review generated code before deploying to production, especially for security-sensitive applications.

## Compliance Skill

The **compliance** skill provides code-level enforcement for four major regulatory frameworks:

### HIPAA
- PHI (Protected Health Information) handling patterns
- Audit logging for data access
- Encryption requirements for data at rest and in transit
- Access control implementation
### SOC 2
- Control implementation patterns (access control, change management, monitoring)
- Audit trail generation
- Incident response procedures
- Vendor risk assessment guidance

### GDPR
- Data subject rights implementation (access, rectification, erasure, portability)
- Consent management patterns
- Data processing agreement templates
- Cross-border data transfer requirements

### PCI DSS
- Cardholder data environment (CDE) scoping
- Encryption and tokenization patterns
- Access control and segmentation
- Logging and monitoring requirements

The compliance skill generates code that implements these requirements, not just documentation. It works best when combined with the **security** skill for vulnerability scanning and **secrets-management** for credential handling.

## Reporting Vulnerabilities

If you discover a security vulnerability in VibeCraft, please report it responsibly:

1. **Do not** open a public GitHub issue for security vulnerabilities.
2. Email **hoodrhysg@gmail.com** with a description of the vulnerability, steps to reproduce, and any relevant context.
3. You will receive an acknowledgment within 48 hours.
4. A fix will be prioritized based on severity and released as a patch version.

We appreciate responsible disclosure and will credit reporters in the changelog (unless you prefer to remain anonymous).