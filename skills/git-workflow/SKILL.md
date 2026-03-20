---
name: git-workflow
description: "Git workflow, conventional commits, branching, PR best practices. Use when asked about git, commit, branch, merge, PR, rebase, cherry-pick, git hooks, monorepo git. Do NOT use for CI/CD pipelines (use cicd) or deployment (use deployment)."
---

# Git Workflow

Smart git practices for solo devs and teams.

## Commit Conventions

Use Conventional Commits:
```
<type>(<scope>): <description>
[optional body]
[optional footer]
```

Types: `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `perf`, `ci`.
Rules: Imperative mood, under 72 chars, body explains WHY, reference issues: `fixes #123`.

## Branching Strategies

### Trunk-Based (Recommended)
- `main` always deployable
- Short-lived feature branches (1-3 days)
- Merge via PR with squash
- Deploy from `main` automatically

### GitHub Flow (Teams)
- Branch from `main`, PR back to `main`
- Environment previews per PR
- Merge when CI passes

### Git Flow (Release-Heavy)
- `main` = production releases only
- `develop` = integration branch
- Feature branches from `develop`

## Common Workflows

### Start a Feature
```bash
git checkout main && git pull origin main
git checkout -b feat/user-profiles
git add -p && git commit -m "feat(users): add profile page"
git push -u origin feat/user-profiles
```

### Resolve Merge Conflicts
```bash
git fetch origin && git rebase origin/main
# Resolve conflicts, then:
git add <files> && git rebase --continue
git push --force-with-lease
```

### Undo Mistakes
- `git reset --soft HEAD~1`: Undo, keep staged
- `git reset HEAD~1`: Undo, keep unstaged
- `git revert <hash>`: Revert (creates new commit)
- `git reflog`: Recover deleted branch

## PR Best Practices

- Small PRs (< 400 lines diff)
- Clear title: conventional commit format
- Description: summary, test plan, screenshots
- Self-review before requesting review
- Squash merge to main
- Delete branch after merge

## Git Hooks

Pre-commit: lint-staged via `.husky/pre-commit`.
Commit-msg: commitlint via `.husky/commit-msg`.
Setup: `npm install -D husky lint-staged @commitlint/cli && npx husky init`.

## Monorepo Patterns

- `git log -- apps/web`: History for one package
- Scope commits: `feat(web):` or `fix(api):`
- Use sparse checkout for large monorepos
- Consider `git worktree` for multi-package work

Read `references/git-commands.md` for complete command reference.
Read `references/branching-strategies.md` for detailed branching patterns.
