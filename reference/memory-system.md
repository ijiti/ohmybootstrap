# Memory System

Claude Code provides per-project auto-memory at `~/.claude/projects/<slug>/memory/`. The index file `MEMORY.md` auto-loads into context; topic files load on demand. This doc explains what to put where.

## Three surfaces for persistent state

| Surface | Location | Scope | Audience |
|---|---|---|---|
| **CLAUDE.md** | repo root + nested in subdirs | Per-project | Agents + humans |
| **Project-local scratch** (`DEVSTUFF.md`, `UISTUFF.md`, `INFRA_THINGS.md`, or similar) | repo root | Per-project | Agents + humans |
| **Auto-memory** (`MEMORY.md` + topic files) | `~/.claude/projects/<slug>/memory/` | Per-project, agent-global | Agent only |

**Rule of thumb:** if a human teammate would benefit, put it in the repo (CLAUDE.md or project-local scratch). If only the agent benefits (user preferences, recurring confusion, session-to-session handoffs), put it in auto-memory.

## Typed memory entries

Auto-memory entries fall into four types. Each type has a different trigger for when to save and how to structure.

### user
What the user's role, goals, preferences, and knowledge are. Helps future sessions tailor behavior.
- **Save when:** you learn any durable detail about the user.
- **Example:** "User is a senior Go developer with limited Kubernetes experience — frame infra explanations in terms of systemd/containerd analogues, not K8s primitives."

### feedback
Guidance the user has given about how to approach work — both corrections and confirmations.
- **Save when:** the user corrects you OR confirms a non-obvious approach worked.
- **Structure:** lead with the rule, then `**Why:**` line (the reason), then `**How to apply:**` line (when the guidance kicks in).
- **Example:** "Integration tests must hit a real database, not mocks. **Why:** a prior incident where mock/prod divergence masked a broken migration. **How to apply:** when writing any test for code that touches persistence."

### project
What's happening in the project that isn't derivable from the code or git log — goals, deadlines, stakeholder constraints.
- **Save when:** you learn who is doing what, why, or by when.
- **Always convert relative dates** to absolute (`"Thursday"` → `"2026-03-05"`).
- **Structure:** fact, then `**Why:**`, then `**How to apply:**`.

### reference
Pointers to external systems — dashboards, ticket queues, docs.
- **Save when:** you learn about a resource the project relies on.
- **Example:** "Pipeline bugs are tracked in Linear project INGEST — check there for context when touching ingest code."

## What NOT to save

- Code patterns, conventions, architecture, file paths — derivable from the code.
- Git history or recent changes — `git log` is authoritative.
- Debugging fixes — the fix is in the code; the commit message has context.
- Anything already in CLAUDE.md.
- Ephemeral task state — use tasks, not memory.

Apply this even when the user asks to save. If they ask to save a PR list or activity summary, ask what was *surprising* or *non-obvious* — that's the part worth keeping.

## MEMORY.md as index

`MEMORY.md` should be an index, not content. Each entry: one line, `- [Title](topic-file.md) — one-line hook`. Truncates at ~200 lines — keep it lean.

## Staleness

Memory is point-in-time, not gospel. Before acting on a recalled fact, verify it's still true by reading the current state. If memory conflicts with current observation, trust current and update memory.
