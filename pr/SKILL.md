---
name: pr
description: Create a branch name, short commit message, and pull request title and description.
---

Inspect the current branch's staged changes, unstaged changes, and untracked files to understand what has changed. Then produce a semantic branch name, a Conventional Commits message, and a pull request title and description.

# Branch prefix conventions

| Prefix | Use for |
|--------|---------|
| `feature/` | new features |
| `fix/` | bug fixes |
| `hotfix/` | urgent production fixes |
| `refactor/` | code cleanup with no behaviour change |
| `perf/` | performance improvements |
| `docs/` | documentation only |
| `test/` | tests only |
| `ci/` | CI/CD pipeline changes |
| `chore/` | maintenance (deps, tooling, config) |

# Commit message — Conventional Commits format

```
<type>(<optional scope>): <short description>
```

- **type** must match the branch prefix (e.g. `feature/` → `feat`, `fix/` → `fix`, `docs/` → `docs`, `chore/` → `chore`, `perf/` → `perf`, `test/` → `test`, `ci/` → `ci`, `hotfix/` → `fix`)
- **scope** is optional; use the affected module, file, or area (e.g. `auth`, `readme`, `api`)
- **description** is lowercase, imperative mood, no trailing period, ≤ 72 chars total

# Pull request description — scale to complexity

Assess the size and impact of the changes, then choose the appropriate level of detail:

**Minor** (trivial edits, single file, obvious intent — e.g. typo fix, small config tweak):
- One-sentence summary. No sections needed.

**Moderate** (a few files changed, self-contained feature or fix):
```
## Summary
- <bullet points describing what changed and why>

## Test plan
- <checklist of what to verify>
```

**Complex** (many files, architectural change, breaking change, or non-obvious motivation):
```
## Summary
<short paragraph: what this does and why>

## Changes
- <key change 1>
- <key change 2>

## Breaking changes
<describe any, or remove this section if none>

## Test plan
- <checklist of what to verify>
```

Use your judgement — err on the side of brevity unless the changes genuinely need more context.
