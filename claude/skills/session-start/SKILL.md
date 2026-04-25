---
name: session-start
description: "Use when starting a new session — restores task continuity, runs basic safety checks, optionally creates a worktree, and prepares the environment."
---

# Session Start

Every session begins with this checklist. Complete each step before starting work.

## Checklist

1. **Restore task continuity** — Read `tasks.jsonl` at the repo root (if present). Cross-check against recent `git log origin/<default-branch>` to verify tasks aren't stale (a prior session may have completed work without updating the file). Restore relevant tasks via `TaskCreate`.

2. **Create a worktree (optional but recommended)** — If the project's CLAUDE.md requires branch-based work (no direct commits to the default branch), create one now:
   - Derive a slug from the user's first request (short, kebab-case).
   - If intent isn't clear yet, use `session-YYYYMMDD-HHMM` as a default.
   - Use whatever worktree script the project ships (`tools/worktree-create.sh` is a common convention), or run `git worktree add` directly.
   - `cd` into the worktree — all subsequent work happens there.

3. **Check open PRs** — Use the project's host tool (`gh pr list`, `tea pr ls`, `glab mr list`) to surface in-flight work.

4. **Unlock credentials (if needed)** — Only required when the session needs secrets. Not required for git, PRs, or code work.

5. **Sync submodules** — `git submodule update --remote` if the repo uses submodules.

6. **Run `date`** — Never guess about day of week, holidays, or market hours.

## Notes

- If your CLAUDE.md specifies a `PATH` or tool-location gotcha, re-read it now.
- If the project runs on hardware that may have rebooted (long-running server, VM), consider a reboot check (`cat /var/run/reboot-required 2>/dev/null` on Debian/Ubuntu, or the equivalent for your OS).
