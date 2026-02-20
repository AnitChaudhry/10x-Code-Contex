# 10x Code Context — Agents Index v1.0.0

> **For orchestrators**: Quick reference for all available agents. Read `agents/<name>.md` for full instructions.

## Agent Registry

| # | Agent | Model | Role | Invocation |
|---|-------|-------|------|-----------|
| 1 | context-builder | opus | Deep codebase scan → .ccs/ index files | After `init` or `refresh` |
| 2 | test-runner | sonnet | Execute tests, track results, auto-fix | After `test` or `fix` |
| 3 | code-auditor | opus | Security, perf, dead code, a11y analysis | After `audit` |
| 4 | git-tracker | sonnet | Branch, PR, merge, diff, sync ops | After any git skill |
| 5 | knowledge-guide | haiku | Methodology guidance, note quality | Advisory — any time |

## Tool Matrix

| Agent | Read | Write | Edit | Glob | Grep | Bash | Task |
|-------|:----:|:-----:|:----:|:----:|:----:|:----:|:----:|
| context-builder | x | x | | x | x | | |
| test-runner | x | x | | | x | x | |
| code-auditor | x | | | x | x | | |
| git-tracker | x | x | | | x | x | |
| knowledge-guide | x | | | | x | | |

## Agent → Skill Mapping

| Agent | Primary Skills | When to Invoke |
|-------|---------------|---------------|
| context-builder | init, refresh | New codebase, stale index, major changes |
| test-runner | test, fix | Test execution, failure diagnosis, auto-fix |
| code-auditor | audit, review | Security review, performance check, code quality |
| git-tracker | branch, pr, merge, diff, sync, log, stash | Any git workflow operation |
| knowledge-guide | — | Proactive guidance on methodology, note quality |

## Parallel Safety

| Agent | Can Run In Parallel | Writes To |
|-------|:------------------:|-----------|
| context-builder | no | .ccs/*.md |
| test-runner | yes (per test suite) | .ccs/task.md (append) |
| code-auditor | yes | stdout only |
| git-tracker | no | .ccs/branches/*.md, .ccs/pulls/*.md |
| knowledge-guide | yes | stdout only |

## For SDK Integration

```python
agents = {
    "context-builder": AgentDefinition(
        description="Deep codebase analysis, generates .ccs/ index files",
        prompt=open("agents/context-builder.md").read(),
        tools=["Read", "Glob", "Grep", "Write"],
        model="opus"
    ),
    "test-runner": AgentDefinition(
        description="Run tests, track results, auto-fix failures",
        prompt=open("agents/test-runner.md").read(),
        tools=["Bash", "Read", "Grep", "Write"],
        model="sonnet"
    ),
    # ... see manifest.json for complete definitions
}
```

---
*10x.in — 10x-code-context v1.0.0*
