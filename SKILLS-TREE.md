# 10x Code Context — Skills Tree v1.0.0

> **For agents**: Read this file first. Match your task to a skill below, then read only `skills/<name>/SKILL.md` for full instructions. Do not read all skills — pick only what you need.

## Quick Reference

| # | Skill | Category | Model | Tags | Depends On | Token Est. | Parallel |
|---|-------|----------|-------|------|-----------|-----------|---------|
| 1 | init | context | opus | index, scan, setup, architecture | — | ~8K | no |
| 2 | status | context | haiku | check, health, staleness | init | ~1K | yes |
| 3 | query | context | haiku | search, preview, lookup | init | ~1.5K | yes |
| 4 | refresh | context | opus | rebuild, update, incremental | init | ~5K | no |
| 5 | plan | workflow | opus | plan, task, dependency | init | ~4K | yes |
| 6 | build | workflow | sonnet | implement, create, feature | init, plan | ~6K | no |
| 7 | fix | workflow | sonnet | debug, bugfix, root-cause | init | ~5K | no |
| 8 | refactor | workflow | opus | restructure, blast-radius | init | ~5K | no |
| 9 | test | quality | sonnet | test, verify, auto-fix | init | ~4K | yes |
| 10 | audit | quality | opus | security, performance, a11y, dead-code | init | ~6K | yes |
| 11 | review | quality | opus | code-review, style, logic | init | ~5K | yes |
| 12 | research | quality | opus | docs, errors, best-practices | — | ~3K | yes |
| 13 | branch | git | sonnet | branch, switch, context-ref | init | ~2K | no |
| 14 | pr | git | opus | pull-request, summary, blast-radius | init, branch | ~4K | yes |
| 15 | merge | git | opus | merge, conflict, resolution | init, branch | ~4K | no |
| 16 | diff | git | opus | diff, impact, dependency-chain | init | ~3K | yes |
| 17 | sync | git | sonnet | pull, push, rebase, conflict | init | ~3K | no |
| 18 | log | git | haiku | history, commits, cross-ref | init | ~1.5K | yes |
| 19 | stash | git | haiku | stash, wip, restore | init | ~1.5K | yes |

## Categories

### Context Management
`init` · `status` · `query` · `refresh`

Core indexing and lookup. Run `init` first on any new codebase — all other skills depend on the `.ccs/` index it generates.

### Workflow
`plan` · `build` · `fix` · `refactor`

Task execution with dependency tracking. `plan` before `build`. `fix` for bugs. `refactor` for structural changes.

### Quality
`test` · `audit` · `review` · `research`

Verification and knowledge. These skills are read-heavy and parallel-safe — run multiple simultaneously.

### Git
`branch` · `pr` · `merge` · `diff` · `sync` · `log` · `stash`

Version control with context awareness. Each generates/updates context reference files in `.ccs/`.

## Agents

| Agent | File | Role | Model | Tools |
|-------|------|------|-------|-------|
| context-builder | `agents/context-builder.md` | Deep codebase analysis, generates .ccs/ index | opus | Read, Glob, Grep, Write |
| test-runner | `agents/test-runner.md` | Run tests, track results, auto-fix failures | sonnet | Bash, Read, Grep, Write |
| code-auditor | `agents/code-auditor.md` | Security, performance, dead code, a11y audits | opus | Read, Glob, Grep |
| git-tracker | `agents/git-tracker.md` | Git workflow — branches, PRs, merges, diffs | sonnet | Bash, Read, Grep, Write |
| knowledge-guide | `agents/knowledge-guide.md` | Methodology guidance, note quality, connections | haiku | Read, Grep |

## Dependency Graph

```
init ──┬──→ status, query, refresh
       ├──→ plan ──→ build
       ├──→ fix, refactor
       ├──→ test ──→ fix (auto-fix loop)
       ├──→ audit, review, diff
       ├──→ branch ──→ pr ──→ merge
       ├──→ sync, log, stash
       └──→ (research has no dependency)
```

## Shared State (`.ccs/` directory)

| File | Written By | Read By |
|------|-----------|---------|
| `project-map.md` | init, refresh | all skills |
| `architecture.md` | init, refresh | plan, build, refactor, audit, review |
| `file-index.md` | init, refresh | query, build, fix, refactor |
| `conventions.md` | init, refresh | build, review, audit |
| `task.md` | all skills (append) | status, log |
| `preferences.json` | init | refresh |
| `branches/*.md` | branch | pr, merge, sync |
| `pulls/*.md` | pr | merge, review |

## For Swarm Orchestrators

- **Discovery**: Read this file (~2K tokens) → pick skill → read that skill (~3-8K tokens)
- **Machine-readable**: See `manifest.json` for programmatic access
- **Coordination**: Filesystem-based via `.ccs/` — no network required
- **Parallel reads**: Any number of agents can read the index simultaneously
- **Write lock**: Only one agent should write to `.ccs/task.md` at a time (append-only)
- **Isolation**: Each skill operates in forked context — no cross-skill state leakage
- **Model tiering**: Use the model column to route to appropriate compute tier

---
*10x.in — 10x-code-context v1.0.0*
