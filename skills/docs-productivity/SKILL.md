---
name: docs-productivity
description: "Generate READMEs, API docs, architecture guides, changelogs. Use for documentation tasks. Do NOT use for application code."
---

# Documentation & Productivity

Good documentation is invisible—it compounds learning and cuts support requests. This skill generates comprehensive, maintainable documentation automatically: READMEs from code, API docs from signatures, architecture guides from structure, and changelogs from commits.

## Process: Generate Documentation

1. **Analyze**: Understand codebase structure, entry points, key modules, test usage patterns
2. **Extract**: Pull function signatures, docstrings, comments, and code examples
3. **Choose type**: README, API docs, architecture guide, changelog, or knowledge base article
4. **Generate**: Create formatted documentation with all sections
5. **Link**: Ensure related docs reference each other
6. **Maintain**: Update alongside code changes, version with releases

## Documentation by Type

### README Generation
- Analyze codebase structure and extracts
- Create overview, quick start, usage guide, contributing section
- Add badges, screenshot/demo, license
- Process: Structure → components → examples → format

### API Documentation (JSDoc / Docstrings)
- Extract function signatures and existing docstrings
- Add @param, @returns, @throws, @example tags
- Generate OpenAPI specs for REST APIs
- Format with proper examples and cross-references

### Architecture Guide
- System diagram showing components and data flow
- Component relationships and dependency order
- Data models and entity relationships
- Technology choices and trade-offs

### Changelog Generation
- Parse commits since last tag
- Categorize by type (feat, fix, refactor, docs)
- Group into sections: Added, Changed, Fixed, Removed
- Generate markdown using Keep a Changelog format

### Knowledge Base Articles
- Document recurring patterns and solutions
- Structure: Problem → Solution → Code Example → When to Use
- Link to related articles and resources
- Create searchable documentation library

## Reference Templates

Read `references/doc-templates.md` for:
- README template with sections and badges
- ADR template (Architecture Decision Record)
- API documentation template
- Changelog format and conventions
- Project handoff template
- Technical specification template
- Runbook template for operations
- Incident postmortem template

## Documentation Best Practices

- **Live**: Keep in sync with code, not separate
- **Linked**: Cross-reference related docs
- **Searchable**: Use clear, consistent titles
- **Examples**: Always show real usage
- **Updated**: Track alongside code changes
- **Honest**: Acknowledge limitations

---

**Next Steps**: Ask Claude to "Generate a README from my codebase" or "Document my architecture."
