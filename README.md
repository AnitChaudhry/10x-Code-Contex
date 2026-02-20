# 10x-code-context

Context engineering middleware for Claude Code. Intelligent codebase indexing, token-efficient file selection, local tracking, testing, auditing, and session persistence — all via local MD files.

## What It Does

This skill plugin sits between your queries and Claude Code. It ensures every interaction has precisely the right context — no wasted tokens exploring files that don't matter, no losing track of what was changed.

**Before this skill:** Claude Code explores your codebase from scratch on every query, reading files it doesn't need, losing context when the window fills up.

**With this skill:** Your codebase is indexed once, only relevant files are read per query, and everything is tracked in local MD files that persist across the session.

## Installation

### npx (recommended)
```bash
npx 10x-code-context init
```

### bun
```bash
bunx 10x-code-context init
```

### Global install
```bash
npm install -g 10x-code-context
ccs init
```

### Manual
```bash
git clone https://github.com/AnitChaudhry/10x-Code-Contex.git
# Copy each skill as its own directory (Claude Code requires one level deep)
for d in 10x-Code-Contex/skills/*/; do
  name=$(basename "$d")
  cp -r "$d" ".claude/skills/ccs-$name/"
done
# Copy shared resources
cp -r 10x-Code-Contex/{agents,templates,references} .claude/skills/_ccs/
```
**Important:** Skills must be at `.claude/skills/<name>/SKILL.md` (one level deep). Do NOT copy into `.claude/skills/ccs/skills/` — Claude Code won't discover nested skills.

## Slash Commands

> **Naming:** When installed via npx/CLI, commands use hyphens: `/ccs-init`, `/ccs-build`, etc.
> When installed as a Claude Code plugin (git clone with `.claude-plugin/`), the plugin namespace adds the prefix automatically: `/ccs-init`, `/ccs-build`, etc.

### Context Management
| Command | Description |
|---------|-------------|
| `/ccs-init` | Deep-research the codebase, generate project-map, architecture, file-index, conventions |
| `/ccs-status` | Show what's indexed, staleness, file counts, token savings estimate |
| `/ccs-refresh` | Rebuild index (full, incremental, or session-based) |
| `/ccs-query [question]` | Preview which files would be selected for a given query |

### Workflow
| Command | Description |
|---------|-------------|
| `/ccs-plan [task]` | Plan a task with full dependency-aware context |
| `/ccs-build [task]` | Create/implement with tracked context and commit-style logging |
| `/ccs-refactor [scope]` | Scope a refactor — identify all affected files and dependencies |
| `/ccs-fix [issue]` | Fix bugs with dependency tracking, root-cause analysis, and verification |

### Testing & Quality
| Command | Description |
|---------|-------------|
| `/ccs-test [scope]` | Run tests, track results locally, suggest and auto-fix failures |
| `/ccs-audit [scope]` | Audit code for security, performance, patterns, accessibility, dead code |
| `/ccs-review [scope]` | Code review with full context — style, logic, security, performance |

### Research & Docs
| Command | Description |
|---------|-------------|
| `/ccs-research [query]` | Search official docs, resolve errors, check deps, find best practices |

### Git Workflow
| Command | Description |
|---------|-------------|
| `/ccs-branch` | Create/switch branches with auto-generated context reference files |
| `/ccs-pr` | Prepare PR with full context — title, summary, blast radius, review areas |
| `/ccs-merge` | Merge with dependency checking — conflict prediction, resolution context |
| `/ccs-diff` | Smart diff with impact analysis — dependency chains, blast radius, categorization |
| `/ccs-sync` | Pull/rebase/push with conflict context and resolution recommendations |
| `/ccs-stash` | Stash with tracked context — remembers what you were working on |
| `/ccs-log` | Smart commit history — groups by branch, cross-references with task.md |

## How It Works

### 1. Initialization (`/ccs-init`)
Scans your entire codebase and generates local reference files in `.ccs/`:
- **project-map.md** — File tree + dependency graph (imports/exports/references)
- **architecture.md** — Tech stack, patterns, entry points, data flow
- **file-index.md** — Files ranked by importance (most-imported = highest rank)
- **conventions.md** — Coding style, naming patterns, test patterns

### 2. Per-Query Context (automatic)
When you ask Claude Code anything, the skill:
1. Looks up the pre-built index for matching files/symbols
2. Runs targeted Glob + Grep (zero API cost) to find candidates
3. Follows import/dependency chains of matched files
4. Claude Code reads ONLY the files that matter

### 3. Session Tracking
Every action is logged in `.ccs/task.md` with git-commit-style entries:
- Task description, files read/modified/created/deleted
- Dependencies identified, status, diff summary
- Persists context without consuming API tokens

## Model Strategy

| Model | Used For | Commands |
|-------|----------|----------|
| Haiku 4.5 | Lightweight lookups, scanning, status checks | status, refresh, query, stash, log |
| Sonnet 4.6 | Standard coding execution | build, fix, test, branch, sync |
| Opus 4.6 | Deep reasoning, architecture, complex analysis | init, plan, refactor, audit, review, research, pr, merge, diff |

## Generated Files

All context files are stored in `.ccs/` (add to `.gitignore`):
```
.ccs/
├── project-map.md      # File structure + dependency graph
├── architecture.md     # Tech stack, patterns, data flow
├── file-index.md       # Files ranked by importance
├── conventions.md      # Coding style and patterns
├── task.md             # Session task log (commit-style)
└── preferences.json    # User preferences (refresh mode, etc.)
```

## Index Refresh Modes

Set your preference on first init (saved in `.ccs/preferences.json`):
- **On-demand** — Run `/ccs-refresh` manually
- **Incremental** — Auto-detect changed files, re-index only those
- **Session-based** — Fresh index at the start of each session

## Swarm Architecture

This skill set is designed as a **standalone skills tree** — reusable building blocks that any agent, terminal, or orchestrator can discover and consume.

### Discovery Flow

```
Agent joins workspace → reads SKILLS-TREE.md (~2K tokens)
  → matches task to skill via tags/category
  → reads only that skill's SKILL.md (~3-8K tokens)
  → executes with isolated context
```

### Index Files

| File | Purpose | Audience |
|------|---------|----------|
| `SKILLS-TREE.md` | Master index — all 19 skills with metadata | Agents (human-readable) |
| `manifest.json` | Machine-readable skill/agent definitions | SDK, orchestrators, CI/CD |
| `AGENTS-INDEX.md` | 5 agents with tool restrictions + parallel safety | Agent coordinators |

### Skill Metadata (in each SKILL.md frontmatter)

Every skill includes enriched YAML frontmatter:
- `category` — context, workflow, quality, git
- `tags` — searchable keywords for matching
- `depends-on` — prerequisite skills
- `input` / `output` — what it expects and produces
- `token-estimate` — approximate token budget
- `parallel-safe` — whether multiple agents can run it concurrently

### For SDK Integration

```python
import json

manifest = json.load(open("manifest.json"))
for skill in manifest["skills"]:
    if "debug" in skill["tags"]:
        print(f"Use {skill['id']} → {skill['path']}")
```

### Coordination

- **Filesystem-based** — no network required, all state in `.ccs/`
- **Parallel reads** — any number of agents can read index simultaneously
- **Write locks** — only one agent writes to `.ccs/task.md` at a time (append-only)
- **Context isolation** — each skill runs in forked context, no cross-skill state leakage

## License

MIT

---
*Built by [10x.in](https://10x.in) — 10x-code-context v1.0.0*
