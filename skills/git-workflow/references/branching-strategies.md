# Branching Strategies

## Trunk-Based Development

Best for: Small teams, solo devs, continuous deployment

```
main ──●──●──●──●──●──●──●──●──
         \     /   \       /
          feat-a    fix-123
```

- `main` is always deployable
- Short-lived feature branches (1-3 days max)
- Merge via PR with squash
- Deploy from `main` automatically
- Requires good testing and CI

## GitHub Flow

Best for: Teams with preview environments, GitHub-focused workflows

```
main ──●──●──●──●──●──●──
         \       /
          feature/user-auth
          (PR + review + merge)
```

- Branch from `main`, PR back to `main`
- Environment previews per PR
- Merge when CI passes and review approved
- Simple and clear for teams
- One branch per feature

## Git Flow

Best for: Release-heavy projects, formal version cycles

```
main    ──●─────────────●──
           \           /
develop ──●──●──●──●──●──
             \   /
              feat-x
```

- `main` = production releases only
- `develop` = integration branch
- Feature branches from `develop`
- Release branches for stabilization
- Hotfix branches for production bugs
- Use when you need formal release cycles

## Release Branch Pattern

For projects with versioned releases:

```
main ──●──●──●──────●──
       (v1.0)(v1.1) (v1.2)
         \      \      \
          rel-1.0 rel-1.1 rel-1.2
            \        \       \
develop ──●──●──●──●──●──●──●──
             |  |  |
           feat-1,2,3...
```

- `rel-X.Y` branch for stabilization
- Cherry-pick critical fixes to both main and develop
- Tag releases on main with semver
- Continue development on develop while release stabilizes

## Feature Flag Pattern

Instead of long-lived branches:

```
main ──●──●──●──●──●── (always deployable, feature behind flag)

FEATURE_X=false → old behavior
FEATURE_X=true → new behavior (for beta testers or percentage rollout)
```

- Deploy without merging PR
- Toggle feature on/off in production
- Gradual rollout to users
- Easy rollback (just flip the flag)
- Works with any branching strategy

## Stacked PRs (for large features)

Break large features into stacked PRs:

```
main ──●──●──●──●──●──●──●── (merge order: base → feature-1 → feature-2 → feature-3)
                 \
                  feature-1: Base scaffolding (PR 1)
                   \
                    feature-2: Core logic (PR 2, depends on PR 1)
                     \
                      feature-3: Polish (PR 3, depends on PR 2)
```

- Each PR is reviewable independently
- Smaller diffs, easier review
- Clear dependency chain
- Merge in order: base first, then dependent features
- Tools: GitHub's draft/ready states, or git-stacked-rebase
