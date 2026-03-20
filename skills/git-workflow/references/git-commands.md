# Git Command Reference

Quick reference for common and advanced git operations.

## Daily Commands

```bash
# Status and info
git status                    # What's changed
git diff                      # Unstaged changes
git diff --staged             # Staged changes
git log --oneline -20         # Recent commits
git log --graph --oneline     # Branch visualization

# Staging
git add <file>                # Stage specific file
git add -p                    # Stage interactively (chunk by chunk)
git add .                     # Stage everything (careful!)
git reset HEAD <file>         # Unstage a file

# Committing
git commit -m "message"       # Commit with message
git commit --amend            # Amend last commit (before push)
git commit --amend --no-edit  # Amend without changing message

# Branching
git branch                    # List local branches
git branch -a                 # List all branches (including remote)
git checkout -b feat/name     # Create and switch to new branch
git switch feat/name          # Switch to existing branch (modern syntax)
git branch -d feat/name       # Delete merged branch
git branch -D feat/name       # Force delete unmerged branch

# Remote
git fetch origin              # Download remote changes (don't merge)
git pull origin main          # Fetch + merge
git pull --rebase origin main # Fetch + rebase (cleaner history)
git push -u origin feat/name  # Push and set upstream
git push --force-with-lease   # Safe force push
```

## Rebase Operations

```bash
# Rebase onto main
git checkout feat/my-feature
git rebase origin/main

# Interactive rebase (squash, reword, reorder)
git rebase -i HEAD~5

# Rebase commands in editor:
# pick   = use commit as-is
# reword = edit commit message
# squash = merge into previous commit
# fixup  = squash but discard message
# drop   = remove commit

# Abort rebase if things go wrong
git rebase --abort

# Continue after resolving conflicts
git add .
git rebase --continue
```

## Cherry-Pick

```bash
# Apply a specific commit to current branch
git cherry-pick <commit-hash>

# Cherry-pick without committing (stage only)
git cherry-pick --no-commit <commit-hash>

# Cherry-pick a range
git cherry-pick <start>..<end>
```

## Stash

```bash
git stash                     # Stash tracked changes
git stash -u                  # Include untracked files
git stash list                # List all stashes
git stash pop                 # Apply latest and remove
git stash apply               # Apply latest but keep in stash
git stash drop stash@{2}      # Remove specific stash
git stash show -p stash@{0}   # View stash contents
```

## Recovery & Undo

```bash
# Undo uncommitted changes to a file
git checkout -- <file>

# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Undo last commit, keep changes unstaged
git reset HEAD~1

# Undo last commit, discard changes entirely
git reset --hard HEAD~1

# Revert a pushed commit (creates new commit)
git revert <commit-hash>

# Find lost commits
git reflog

# Recover file from specific commit
git checkout <commit-hash> -- <file>
```

## Investigation

```bash
# Who wrote each line
git blame <file>

# Search commit messages
git log --grep="search term"

# Search code changes
git log -S "function_name"    # When was this string added/removed?
git log -p -- <file>          # Full diff history of a file

# Find when a bug was introduced
git bisect start
git bisect bad                # Current commit has the bug
git bisect good <old-hash>    # This commit was good
# Git checks out middle commit, you test, mark good/bad, repeat
git bisect reset              # When done
```

## Cleanup

```bash
# Remove branches already merged to main
git branch --merged main | grep -v main | xargs git branch -d

# Remove remote tracking branches that no longer exist
git remote prune origin

# Clean untracked files (careful!)
git clean -fd                 # Remove untracked files and dirs
git clean -fdn                # Dry run (show what would be deleted)

# Reduce repo size
git gc --aggressive
```

## GitHub CLI (gh)

```bash
# PRs
gh pr create --title "feat: ..." --body "..."
gh pr list
gh pr view 123
gh pr checkout 123
gh pr merge 123 --squash --delete-branch

# Issues
gh issue create --title "Bug: ..." --body "..."
gh issue list --assignee @me
gh issue close 456

# Releases
gh release create v1.0.0 --generate-notes

# Actions
gh run list
gh run view 789
gh run rerun 789
```

## Git Config Essentials

```bash
# Identity
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Default branch name
git config --global init.defaultBranch main

# Auto-rebase on pull
git config --global pull.rebase true

# Useful aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --graph --oneline --all"
```
