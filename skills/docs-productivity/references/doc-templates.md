# Documentation Templates Reference

Ready-to-use templates for every documentation type. Copy, customize, and publish.

## README Template

```markdown
# [Project Name]

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![npm version](https://badge.fury.io/js/%40[scope]%2F[name].svg)](https://badge.fury.io/js/%40[scope]%2F[name])
[![CI Status](https://github.com/[owner]/[repo]/workflows/CI/badge.svg)](https://github.com/[owner]/[repo]/actions)
[![Code Coverage](https://img.shields.io/codecov/c/github/[owner]/[repo])](https://codecov.io/gh/[owner]/[repo])

[One-sentence description of what your project does]

## Features

- Feature 1
- Feature 2
- Feature 3

## Quick Start

### Installation

```bash
npm install [project-name]
```

### Basic Usage

```javascript
import { [function] } from '[project-name]';

const result = [function](input);
console.log(result);
```

## Usage Guide

### Common Tasks

#### Task 1: [Task Description]

```javascript
// Code example
const output = [function]({ option: 'value' });
```

#### Task 2: [Task Description]

```javascript
// Code example
```

### Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| option1 | string | 'default' | What this does |
| option2 | boolean | false | What this does |

### Error Handling

```javascript
try {
  const result = await [function](input);
} catch (error) {
  console.error('Error:', error.message);
}
```

## API Reference

### [FunctionName]

Detailed description of what this function does.

**Parameters:**
- `param1` (string): Description of param1
- `param2` (number, optional): Description of param2

**Returns:** Description of return value

**Throws:** Description of possible errors

**Example:**
```javascript
const result = [functionName](param1, param2);
```

### [ClassName]

**Methods:**
- `.method1()`: Description
- `.method2()`: Description

## Contributing

### Setup

```bash
git clone [repo]
cd [project]
npm install
npm run dev
```

### Testing

```bash
npm test
```

### Code Style

This project uses Prettier and ESLint. Run:
```bash
npm run lint:fix
```

### Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Architecture

[Brief overview of major components]

```
Client → API Server → Database
           ↓
         Cache Layer
```

For detailed architecture, see [ARCHITECTURE.md](./ARCHITECTURE.md).

## Troubleshooting

### Issue: [Common Issue]

**Solution:** [How to fix it]

### Issue: [Another Common Issue]

**Solution:** [How to fix it]

## Performance

[Key performance characteristics and benchmarks if applicable]

## License

MIT License - see [LICENSE](./LICENSE) file for details

## Maintainers

- [Name] (@github-handle)
- [Name] (@github-handle)

## Acknowledgments

- [Acknowledgment 1]
- [Acknowledgment 2]

## Related Projects

- [Related Project 1](link)
- [Related Project 2](link)
```

## ADR Template (Architecture Decision Record)

```markdown
# ADR [000]: [Decision Title]

**Status:** Accepted | Pending | Superseded | Deprecated

**Date:** 2026-03-19

## Context

[Describe the issue or question that prompted this decision. What is the background?]

Example: We need to choose a database for our new service. Our system handles high-volume writes and requires horizontal scalability.

## Decision

[Clearly state what was decided.]

Example: We will use PostgreSQL for primary data storage with Redis for caching.

## Rationale

[Explain why this decision was made. What drove the choice?]

Example:
- PostgreSQL: ACID guarantees, relational model matches our data, mature ecosystem
- Redis: Proven for caching, fast reads, simple to operate
- Alternative 1 (MongoDB): Rejected because our data is relational and we don't need document flexibility
- Alternative 2 (MySQL): Rejected because PostgreSQL has better JSON support

## Consequences

### Positive
- Improved query performance with proper indexes
- Strong consistency guarantees
- Easy to scale read replicas

### Negative
- Increased operational complexity
- Need to maintain database
- Cost for managed hosting

### Trade-offs
- Chose consistency over eventual consistency
- Accepted higher operational burden for data reliability

## Alternatives Considered

### Alternative 1: MongoDB
- **Pros:** Flexible schema, easy to scale horizontally
- **Cons:** No ACID transactions (at the time), relational model not needed
- **Outcome:** Rejected

### Alternative 2: DynamoDB
- **Pros:** Fully managed, automatically scales
- **Cons:** Query limitations, higher cost, vendor lock-in
- **Outcome:** Rejected

## Implementation Plan

1. Set up PostgreSQL cluster (Phase 1)
2. Migrate existing data (Phase 2)
3. Deploy application with dual writes (Phase 3)
4. Validate correctness (Phase 4)
5. Switch to PostgreSQL-only (Phase 5)

## Related Decisions

- ADR-001: Service Architecture (foundational)
- ADR-003: Caching Strategy (builds on this)

## Reviewed By

- [Name] - Architecture Review
- [Name] - Operations Review

## See Also

- [Related documentation]
- [Implementation issue link]
```

## API Documentation Template

```markdown
# API Documentation

## Overview

[Brief description of the API]

## Authentication

[How to authenticate requests]

Example:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com/endpoint
```

## Base URL

```
https://api.example.com/v1
```

## Error Handling

All errors return JSON with this structure:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": { "field": "additional info" }
  }
}
```

## Endpoints

### Get User

**Endpoint:** `GET /users/{id}`

**Description:** Retrieve a user by ID

**Parameters:**
| Name | Type | Location | Required | Description |
|------|------|----------|----------|-------------|
| id | string | path | Yes | User ID |
| fields | string | query | No | Comma-separated fields to include |

**Response (200 OK):**
```json
{
  "id": "user_123",
  "name": "John Doe",
  "email": "john@example.com",
  "createdAt": "2026-01-01T00:00:00Z"
}
```

**Response (404 Not Found):**
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

**Example Request:**
```bash
curl https://api.example.com/v1/users/user_123
```

### Create User

**Endpoint:** `POST /users`

**Description:** Create a new user

**Request Body:**
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "secure_password"
}
```

**Response (201 Created):**
```json
{
  "id": "user_124",
  "name": "Jane Doe",
  "email": "jane@example.com",
  "createdAt": "2026-03-19T12:00:00Z"
}
```

**Example Request:**
```bash
curl -X POST https://api.example.com/v1/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Doe","email":"jane@example.com"}'
```

## Rate Limiting

- 1000 requests per hour per API key
- Rate limit info in response headers:
  - `X-RateLimit-Limit`: Total requests allowed
  - `X-RateLimit-Remaining`: Requests remaining
  - `X-RateLimit-Reset`: Unix timestamp when limit resets

## SDK Examples

### JavaScript

```bash
npm install example-api-sdk
```

```javascript
import { ApiClient } from 'example-api-sdk';

const client = new ApiClient('YOUR_API_KEY');
const user = await client.users.get('user_123');
```

### Python

```bash
pip install example-api-sdk
```

```python
from example_api import ApiClient

client = ApiClient('YOUR_API_KEY')
user = client.users.get('user_123')
```
```

## Changelog Format (Keep a Changelog)

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Dark mode support
- New API endpoint for batch operations

### Changed
- Improved performance of search queries

### Fixed
- Bug in form validation with special characters

## [1.2.0] - 2026-03-19

### Added
- User authentication via OAuth 2.0
- Dark mode preference in settings
- API rate limiting

### Changed
- Updated dependencies to latest versions
- Improved error messages to be more helpful

### Fixed
- Memory leak in WebSocket connection
- Incorrect pagination in list endpoints
- Race condition in cache invalidation

### Security
- Updated TLS configuration to enforce 1.2+
- Added input sanitization for all user inputs

## [1.1.0] - 2026-03-10

### Added
- User profile pages
- Email notifications
- Search functionality

### Fixed
- Missing validation on form inputs
- Incorrect calculation in billing

## [1.0.0] - 2026-02-28

Initial release
```

## Project Handoff Template

```markdown
# Project Handoff Documentation

## Project Overview

**Name:** [Project Name]
**Owner:** [Previous Owner Name]
**Handoff Date:** 2026-03-19
**New Owner:** [New Owner Name]

[1-2 paragraph overview of project]

## Quick Start

See [Development Setup](#development-setup) below.

## Architecture

[High-level overview of system architecture]

### System Diagram

```
[ASCII or description of system components]
```

### Key Components

1. **Frontend**: React app at `/frontend`
2. **Backend API**: Node.js service at `/backend`
3. **Database**: PostgreSQL
4. **Cache**: Redis

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed architecture.

## Development Setup

### Prerequisites
- Node.js 18+
- PostgreSQL 14+
- Redis 6+

### Installation

```bash
git clone [repo]
cd [project]
npm install
npm run setup
npm run dev
```

### Local Database

```bash
# Create local database
createdb [project]_dev

# Run migrations
npm run migrate

# Seed data (optional)
npm run seed
```

## Testing

```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage
```

## Deployment

### Staging

```bash
git push origin main  # Triggers auto-deploy to staging
```

### Production

```bash
npm version patch  # Updates version
git push origin main
git push origin --tags
# Verify deployment at https://prod.example.com
```

### Rollback

```bash
git revert [commit-hash]
git push origin main
```

## Monitoring & Operations

### Logs

- **Application**: Cloudwatch at [link]
- **Database**: [Database monitoring tool]
- **Errors**: Sentry at [link]

### Alerts

- Configured in Datadog: [link to dashboard]
- Alert recipients: [email/slack channel]

### Common Issues

**Issue: API returns 500 errors**
1. Check application logs
2. Check database connections
3. Restart service: `npm run restart`

**Issue: Database is slow**
1. Check slow query log
2. Review indexes
3. Run `ANALYZE` on tables

See [Runbook.md](./RUNBOOK.md) for more operational procedures.

## Team Contacts

- **Tech Lead:** [Name] - [email/slack]
- **On-Call:** [Rotation link]
- **Product Manager:** [Name] - [email/slack]

## Current Status

### Known Issues

- [Issue 1]: Workaround: [workaround]
- [Issue 2]: Scheduled fix: [date]

### In Progress

- [Feature 1] (ETA: date)
- [Feature 2] (ETA: date)

### Roadmap

See [ROADMAP.md](./ROADMAP.md) for upcoming features and timeline.

## Key Files

| File | Purpose |
|------|---------|
| `.env.example` | Environment variables template |
| `src/api.ts` | API routes and handlers |
| `src/db.ts` | Database setup and queries |
| `src/middleware/` | Express middleware |
| `tests/` | Test files |

## Architecture Decisions

All major decisions documented in [ADRs](./docs/adr/):
- ADR-001: Technology Stack
- ADR-002: Authentication Strategy
- ADR-003: Caching Strategy

## Technical Debt

- [Debt item 1]: Effort: medium, Priority: low
- [Debt item 2]: Effort: large, Priority: medium

See [TECHNICAL_DEBT.md](./TECHNICAL_DEBT.md) for full list.

## Questions?

- [Link to documentation]
- [Link to discussion board]
- Reach out to [previous owner name]
```

## Technical Specification Template

```markdown
# Technical Specification: [Feature Name]

**Date:** 2026-03-19
**Author:** [Your Name]
**Status:** Draft | Review | Approved

## Overview

[1-2 paragraph overview of feature]

## Goals

- Goal 1: [What success looks like]
- Goal 2: [What success looks like]

## Requirements

### Functional Requirements

- Requirement 1: [User can do X]
- Requirement 2: [System shall handle Y]

### Non-Functional Requirements

- Performance: [Criteria]
- Scalability: [Criteria]
- Security: [Criteria]

## Design

### Architecture

[Describe how this feature is implemented architecturally]

### Data Model

[Entity relationships if applicable]

### API Changes

[New endpoints or changes to existing ones]

### Implementation Plan

1. Phase 1: [Setup/foundation]
2. Phase 2: [Core implementation]
3. Phase 3: [Testing and refinement]
4. Phase 4: [Deployment]

## Testing Strategy

- Unit tests: [Coverage target]
- Integration tests: [Key scenarios]
- Performance tests: [Benchmarks]

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| [Risk] | [Impact] | [Mitigation plan] |

## Timeline

- Design review: [Date]
- Implementation: [Date range]
- QA: [Date range]
- Release: [Date]
```

## Runbook Template (Operations)

```markdown
# Runbook: [System Name]

## Overview

[Description of system and its purpose]

## Common Tasks

### Starting the Service

```bash
systemctl start [service-name]
systemctl status [service-name]
```

### Checking Health

```bash
curl https://api.example.com/health
```

Response should be:
```json
{ "status": "ok" }
```

### Viewing Logs

```bash
journalctl -u [service-name] -f
```

## Troubleshooting

### Problem: Service Won't Start

**Symptoms:** Service fails to start with error message

**Investigation:**
1. Check logs: `journalctl -u [service-name] -n 50`
2. Check dependencies: `systemctl status [dependency]`
3. Verify configuration: `cat /etc/[service]/config.yaml`

**Resolution:**
[Steps to fix]

### Problem: High CPU Usage

**Symptoms:** CPU at 100%, service slow

**Investigation:**
1. Identify process: `top -p [pid]`
2. Check connections: `netstat | grep [service]`
3. Check database queries: [DB tool]

**Resolution:**
[Steps to fix]

## Emergency Procedures

### Service Outage

1. Check status: `systemctl status [service-name]`
2. Restart: `systemctl restart [service-name]`
3. Check logs: `journalctl -u [service-name] -f`
4. If still down, escalate to [team]

### Database Issues

1. Connect to database: `psql [dbname]`
2. Check connections: `SELECT count(*) FROM pg_stat_activity;`
3. If locked, terminate query: `SELECT pg_terminate_backend(pid);`

## Escalation

- **Level 1:** [Team contact]
- **Level 2:** [Manager]
- **Level 3:** [Director]

## Maintenance Windows

- Scheduled maintenance: [Day/Time]
- Deployment window: [Time window]
```

## Incident Postmortem Template

```markdown
# Incident Postmortem: [Incident Name]

**Date:** 2026-03-19
**Duration:** 2 hours (14:00-16:00 UTC)
**Severity:** Critical | High | Medium | Low

## Summary

[Brief description of what happened and impact]

## Timeline

| Time | Event |
|------|-------|
| 14:00 | Alert triggered: Database CPU high |
| 14:05 | On-call engineer acknowledged |
| 14:15 | Root cause identified: N+1 queries |
| 14:45 | Fix deployed |
| 15:00 | Service fully recovered |
| 16:00 | All monitoring normal |

## Root Cause

[Detailed explanation of what caused the incident]

## Impact

- **Users Affected:** 1000+
- **Duration:** 1 hour
- **Revenue Impact:** [If applicable]

## Contributing Factors

- [Factor 1]
- [Factor 2]
- [Factor 3]

## What Went Well

- Quick detection via alerts
- On-call response within 5 minutes
- Communication to stakeholders

## What Went Poorly

- N+1 query wasn't caught in code review
- No performance tests for new endpoint
- Database didn't auto-scale

## Action Items

| Action | Owner | Due Date | Priority |
|--------|-------|----------|----------|
| Add query performance test | [Name] | 2026-03-22 | High |
| Review code review process | [Name] | 2026-03-26 | High |
| Set up database auto-scaling | [Name] | 2026-04-02 | Medium |

## Prevention

- Implement query profiling in CI
- Add performance benchmarks to code review
- Enable database monitoring alerts

## Resources

- [Incident tracking issue]
- [Code review of problematic code]
- [Database logs during incident]
```

---

**Next Steps**: Choose a template matching your current documentation need. Customize with your project details. Build comprehensive docs incrementally.
