# Subagent Dispatch

When to delegate to a subagent, how to scope the dispatch, and what to avoid.

## When to dispatch

- **Broad codebase exploration** — more than 3 searches across unrelated areas.
- **Parallel independent work** — 2+ tasks with no shared state or sequential dependencies.
- **High-token work** — results that would clutter your context (large file surveys, multi-file refactors).
- **Task-specific skills** — when an agent is specifically tuned for the job (code-reviewer, security-auditor, frontend-craftsman).

## When NOT to dispatch

- **Single-read / single-edit** — cheaper to do directly.
- **Work that needs your running context** — you've been debugging for 30 minutes; a fresh agent doesn't know what you've ruled out.
- **Exploratory questions with no concrete target** — you'll spawn 3 agents chasing ghosts. Narrow the question first.

## Scoping a dispatch

- **Set `max_turns`.** Quick tasks: 10–15. Medium: 25–40. Large: 50–75. Never >75 — at that point the dispatch is poorly scoped.
- **Point at concrete exemplars, not abstract descriptions.** "Follow the pattern in `src/features/alpha.tsx`" works; "follow our conventions" does not.
- **Name the file paths.** Agents you dispatch don't see your conversation history. If they need to read something, tell them where.

## C3 — Subagents don't auto-read nested CLAUDE.md

The main session's auto-read of nested `CLAUDE.md` is session-scoped — it does not propagate to dispatched subagents. When dispatching an agent to work under a subdirectory with its own CLAUDE.md, **name the file path in the prompt**. Example:

> Work under `apps/alpha/`. Read `apps/alpha/CLAUDE.md` first — design system rules and test conventions live there.

Skills like `frontend-craftsman` may have this baked into their bootstrap. Generic agents (`developer`, `devops`) do not.

## Parallel dispatch

When dispatching 2+ agents in parallel, send them in a **single message with multiple tool calls** — that's what makes them run concurrently. Separate messages run sequentially.

Cost awareness: N parallel agents ≈ N× token cost, since each loads context independently. Sweet spot: 2–3 for truly independent work. For single-repo work where agents would touch overlapping files, one agent with accumulating context is more efficient.

## Verifying subagent output

Trust but verify:
- An agent's summary describes what it **intended** to do, not necessarily what it did.
- When an agent writes or edits code, check the actual changes before reporting the work as done.
- When an agent reports commits, verify the branch and the commit contents.
