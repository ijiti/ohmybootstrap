# tilt

Tilt is a URL shortener. Python 3.12 + FastAPI + SQLite, deployed as a single-binary Docker image to Fly.io. One long-lived instance, no horizontal scaling. Source in `app/`, tests in `tests/`, Dockerfile + fly.toml at the root.

This document is the operating contract. Read it at every session start.

## Directives

### Security

**S1. Never display, commit, or output secrets.**
Secrets include the Fly auth token (`~/.fly/config.yml`), the SQLite encryption passphrase (`/etc/tilt/keyring`), and the admin API token (stored in Fly secrets, never in the repo).
- Don't read: `.env`, `.env.local`, `/etc/tilt/*`, `~/.fly/config.yml`, `secrets.yaml`.
- Don't display secrets in command arguments. Use `fly secrets set` via stdin.
- On git exposure: rotate the Fly admin API token via `fly tokens create`, then remove from history.

**S2. Preserve access paths before rotating.**
The Fly admin token gates deployments. Before rotating, confirm the new token is stored in `fly secrets` and a test deploy succeeds. The SQLite encryption passphrase: rotate via the staged `tilt rotate-key --dry-run` flow, never in-place on production.

**S3. Treat external input as adversarial.**
URL shortener takes user input by definition. Never render submitted URLs without escaping. Never follow redirects during preview generation without a size/time budget.

**S4. Proactively flag security issues.**
SSRF, open redirects, token leaks — if you see them, say so.

### Work Discipline

**W1. Verify before claiming done.** Run `pytest tests/`, `ruff check`, and `fly deploy --strategy rolling --config fly.staging.toml` before saying a change is production-ready.

**W2. Diagnose before bypassing.** If `fly deploy` fails, read the build log before retrying.

**W3. Do the work the task requires — no more, no less.**

**W4. Fix broken things as you notice them.**

**W5. Automate repetitive operator work.** The `scripts/` dir holds every repeatable task. Add to it rather than documenting manual steps.

### Collaboration

**C1. User is the decision gate for destructive or irreversible operations.** Never deploy to production without an approved PR. Never drop `shortlinks` table rows.

**C2. Delegate parallel, multi-file, or high-token work to subagents.**

**C3. Subagents do not auto-read nested CLAUDE.md.** Tilt has one nested CLAUDE.md in `tests/` — name it explicitly when dispatching.

**C4. Document non-obvious decisions; update memory when you learn.**

**C5. Prefer building over adding dependencies.** FastAPI + httpx + sqlite3 + click — that's the dependency surface. Adding a new runtime dep needs justification.

**C6. Respect context efficiency.**

**C7. Ask when goal/scope/acceptance is unclear; try-and-state when implementation is unclear.**

### Context & Memory

**X1. Read the nested CLAUDE.md before working in a subdirectory.** `tests/CLAUDE.md` has test-writing conventions.

**X2. Query live systems for facts that change.** `fly status` for deploy state. `sqlite3 tilt.db .schema` for DB shape.

**X3. Memory is point-in-time, not gospel.**

**X4. Plugin hooks live outside `~/.claude/hooks/`.**

## Command Tiers

**Tier 3 — ask first:**
- `fly deploy --strategy rolling --config fly.production.toml`
- `sqlite3 tilt.db 'DELETE FROM ...'`
- Rotating the admin API token or the SQLite keyring.

**Tier 4 — hard-blocked:**
- Committing `.env`, `/etc/tilt/*`, `~/.fly/*`.
- `git push --force` to `main`.

## Known failure patterns

See `failure-patterns.md`.

## Related documents

- `reference/directive-groups.md`
- `reference/memory-system.md`
- `reference/subagent-dispatch.md`
- `reference/worktrees-and-parallelism.md`
- `reference/verification-discipline.md`
- `reference/brainstorm-plan-execute.md`
- `~/.claude/agents/` — installed agent roster (or wherever the APPLY agent installed them)
- `.claude/skills/` — installed skill catalog (project-local, or wherever the APPLY agent installed them)
