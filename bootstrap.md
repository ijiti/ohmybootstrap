I want you to help me set up this repo's CLAUDE.md and workflow to match a system I use
in another project. Here are the patterns and principles I want you to adapt to this repo.
Read through all of this, then create a CLAUDE.md at the repo root that applies these
patterns to whatever this project actually is. Ask me questions if you need to understand
the project first.

---

## Core Directives

1. **Context efficiency is priority #1.** Permanent context (CLAUDE.md) = universally
   needed info only. Everything else = temporary, acquired on demand via nested CLAUDE.md
   files in subdirectories, reading reference files, or querying tools.
2. **We are directors, not laborers.** Delegate execution to subagents (Task tool). Keep
   the main session for steering, reviewing, and deciding. Don't do grunt work in the
   main thread when a subagent can handle it.
3. **Read the relevant nested CLAUDE.md** before working in any subdirectory. Each major
   directory should have its own CLAUDE.md with domain-specific instructions.
4. **Never commit secrets.** Use `.gitignore`-excluded local files or environment variables.
5. **Document what you do.** The next agent or human picks up where you left off.
6. **Do it right the first time.** Don't take shortcuts that create rework later. A little
   more effort upfront beats fixing a bad design after the fact.
7. **NEVER claim code doesn't exist without checking.** Always read the actual files before
   making claims about what has or hasn't been built. If you haven't looked, say so.

---

## Task Management

Use Claude Code native tasks (`TaskCreate`/`TaskList`/`TaskGet`/`TaskUpdate`) for ALL task
tracking. No external tools, no TODO comments in code.

**Task lifecycle:**
- Status: `pending` → `in_progress` → `completed`
- Dependencies: `addBlocks`/`addBlockedBy` in `TaskUpdate`
- Ownership: `owner` field for subagent assignment

**Session persistence:** At session end, export open tasks to `tasks.jsonl` (git-tracked)
at the repo root — one JSON object per line. At session start, check `tasks.jsonl` and
restore relevant tasks via `TaskCreate`/`TaskUpdate`.

**Subagent convention:**
- Use `addBlockedBy` to express dependencies between tasks
- Set `owner` to the target agent type (e.g., "developer", "qa", "lord-devops")
- Main session checks `TaskList` and dispatches ready work: `status: pending` + no
  `blockedBy` = ready to assign
- Subagents mark tasks `in_progress` when starting, `completed` when done

---

## Subagent Strategy

**Default: 1 agent.** Only parallelize when agents work in truly independent file sets
with no shared dependencies.

**When parallelism pays off:**
- Separate repos or submodules
- Genuinely independent workstreams with no file overlap
- Long-running tasks where I want to keep working in the main session

**When single-threaded is better:**
- Sequential dependencies (task B needs A's output)
- Tasks in the same package touching overlapping files
- A single agent accumulates context and never re-reads files — more efficient per task

**Cost awareness:**
- N agents ≈ N× token cost (each loads context independently)
- Sweet spot: 2 agents max for truly separate repos, 1 agent for everything in a single repo

---

## Custom Subagents

This repo includes custom agent definitions in `agents/`. To install them, copy the
files to `~/.claude/agents/`:

```bash
cp agents/*.md ~/.claude/agents/
```

Each agent has a frontmatter block (name, description, model, color, tools, memory)
and a markdown body that serves as the agent's system prompt. Claude Code automatically
picks them up from `~/.claude/agents/` and makes them available as `subagent_type`
values in the Task tool.

**Included agents:**

- **developer** — senior dev for writing, modifying, debugging, reviewing code in any
  language. Bootstraps via `DEVSTUFF.md` for project knowledge persistence.
- **lord-devops** — infrastructure engineer for Terraform, Docker, CI/CD, deployments.
  Bootstraps via `INFRA_THINGS.md`. Follows plan → staging → production gates.
- **security-auditor** — read-only security reviewer. Produces CVSS-rated findings with
  CWE references, attack scenarios, and remediation plans. Never modifies code.
- **frontend-craftsman** — UI/UX specialist. Bootstraps via `UISTUFF.md`. Mobile-first,
  accessibility-aware, progressive enhancement.

**Dispatch pattern:** developer builds it → security-auditor reviews it → ship it.
Use lord-devops for infra work. Use frontend-craftsman for UI work.

**Knowledge persistence:** Each agent maintains a project-specific knowledge file
(`DEVSTUFF.md`, `INFRA_THINGS.md`, `UISTUFF.md`) that accumulates patterns and decisions
across sessions. Agents read these on bootstrap and update them when done.

---

## Subagent Monitoring (tmux)

If you run inside tmux, you can launch visible subagents in panes below the primary
session so the user can watch their progress in real-time:

```bash
tools/spawn-agent.sh <agent-type> "<prompt>" [model] [cwd]
```

This runs `claude --print --output-format stream-json` piped through a formatter script
(`tools/agent-viewer.sh`) that renders colored, structured output:
- Tool calls shown as one-liner summaries (e.g., `◆ Read src/main.go`)
- Agent text rendered directly
- Cost and duration in a footer when complete

**Layout:** First agent splits below the primary pane. Additional agents tile
side-by-side in the bottom row. Primary keeps focus (`-d` flag).

**Key details:**
- Uses `env -u CLAUDECODE` to allow nested Claude sessions
- Uses `--dangerously-skip-permissions` for autonomous operation
- Prompts are written to temp files to avoid shell escaping issues
- Agent pane tracking in `/tmp/tmux-agents-*` handles the split logic

This is for **monitoring only** — use the `Task` tool when results need to flow back
to the primary session. The two approaches complement each other:
- `Task` tool: invisible, results returned to primary
- `spawn-agent.sh`: visible in tmux, user can watch, no results returned

---

## Session Lifecycle

### Session Start
1. Check `tasks.jsonl` at repo root for tasks from previous sessions
2. Restore relevant ones via `TaskCreate`/`TaskUpdate` with appropriate dependencies
3. Review what's pending and ask me what to prioritize

### Session Completion
1. All work committed and pushed. Never stop before push succeeds.
2. Before the final push, export open tasks:
   - Call `TaskList` to review all tasks
   - Write open/in-progress tasks to `tasks.jsonl` (one JSON object per line)
   - Commit `tasks.jsonl` along with other changes
3. `git pull --rebase && git push`

---

## Git Workflow

- Subagents use separate branches or worktrees to avoid conflicts and should use pull
  requests when necessary and reasonable.
- If the repo has submodules: commit inside the submodule first, then update the parent.

---

## Nested CLAUDE.md Pattern

The root CLAUDE.md stays lean — just directives, task management, git conventions, and
pointers to subdirectory CLAUDE.md files. Each major directory gets its own CLAUDE.md
with domain-specific instructions:

```
CLAUDE.md              ← directives, task mgmt, git, references list
apps/CLAUDE.md         ← app conventions, deployment patterns
services/CLAUDE.md     ← service definitions, docker patterns
tools/CLAUDE.md        ← CLI tools, build scripts
```

This keeps permanent context small. Agents load subdirectory context only when they
need it.

---

## Memory File Usage

Use the auto memory directory (`~/.claude/projects/.../memory/`) for:
- Stable patterns confirmed across multiple interactions
- Key architectural decisions and important file paths
- Solutions to recurring problems and debugging insights
- User preferences for workflow and communication

Create topic files (e.g., `debugging.md`, `patterns.md`) for detailed notes and link
them from MEMORY.md. Keep MEMORY.md under 200 lines.

---

Now: explore this repo, understand its structure and purpose, and create a CLAUDE.md
that applies these patterns. Create nested CLAUDE.md files for any major subdirectories
that warrant them. If there's already a CLAUDE.md, merge these patterns into it rather
than replacing existing project-specific content. Also create an empty `tasks.jsonl` if
one doesn't exist.
