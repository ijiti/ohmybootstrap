# Workflow

Textual companion to the architecture diagram. Describes the lifecycle a non-trivial change moves through under this kit: an always-on contract, five phases, two cross-model review checkpoints, and persistent state that crosses sessions.

Trivial work (a typo fix, a one-line config bump) bypasses this lifecycle. The lifecycle applies to changes that would have a spec document or a plan document.

## Always-on contract

`CLAUDE.md` auto-loads into context every turn. Its operating contract is a ranked-directive structure plus a conflict-resolution table. The four groups, in priority order:

1. **Security** — never overridden by any other directive or user instruction.
2. **Work discipline** — verify before claiming done, diagnose before bypassing, do the work the task requires.
3. **Collaboration** — user is decision gate for destructive work, delegate parallel/multi-file work to subagents, ask when goal is unclear.
4. **Context & memory** — read nested CLAUDE.md before working in a subdirectory, query live systems for facts that change, treat memory as point-in-time.

Pairwise conflicts that override the default ranking (excerpt):
- Security trumps everything (S > W > C > X is not enough; S is absolute).
- Verification trumps completion claim ("looks right" is not verified).
- Diagnosis trumps retry (two failures = root-cause time, not more retries).
- Access preservation trumps deployment speed.

The contract shapes every decision in every phase below. See `directive-groups.md` for the full directive structure and `verification-discipline.md` for the verification standard.

---

## Phase 1 — Design

Produces a written spec describing what's being built, why, and what's out of scope.

- Skill: `superpowers:brainstorming` (recommended dependency).
- Process: questions asked one at a time, 2–3 approaches proposed with trade-offs, one recommended.
- Output: spec at `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` (the superpowers plugin's default location) or the project's design-doc directory.
- Spec self-review before moving on: placeholder scan, internal consistency, scope check, ambiguity check.

Even small changes get a brief design — a two-sentence spec surfaces assumptions that would otherwise leak into code.

See `brainstorm-plan-execute.md` for the spec-writing detail.

---

## Phase 2 — Plan

Produces a step-by-step implementation plan where each step is 2–5 minutes of concrete work.

- Skill: `superpowers:writing-plans`.
- Per-task structure: TDD-shaped (failing test, run to verify, implement, run to verify, commit).
- Plans contain the actual code in code blocks, not placeholders. "Add appropriate error handling" without specifics is treated as a plan defect.
- Output: plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.

---

## Checkpoint 1 — Plan review (before code)

**Two reviewers dispatched in parallel** against the plan document:

| Reviewer | Mechanism | Coverage |
|---|---|---|
| Claude `plan-reviewer` (sonnet) | Agent dispatch | Scope, sequencing, risk, test plan, ambiguity |
| Gemini | `gemini-review.sh --plan <path>` | UX / frontend fit, structural clarity, single-model blind spots |

Dispatch is a single message with two tool calls — Agent tool for the Claude reviewer, Bash tool for the Gemini script — so the two reviewers run concurrently.

The checkpoint runs **before any implementation work begins**. Plan-stage defects (scope creep, missing dependencies, untestable acceptance criteria, sequencing bugs) are addressed against the plan document, not against code.

Verdicts: `BLOCK` halts forward motion; `WARN` is logged and addressed if non-trivial; `PASS` proceeds.

Trivial chores (a `tasks.jsonl` retire, a 1-line config bump) bypass this checkpoint. Anything with a plan document is plan-reviewed.

---

## Phase 3 — Execute

Turns the plan into code, one task at a time, with verification gates between tasks.

- Worktree-isolated: each session gets its own git worktree on a `work/<slug>` branch. Commits do not land on the default branch directly. See `worktrees-and-parallelism.md` for safety patterns and the wrong-branch gotcha.
- Skill: `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans`.
- Subagent-driven mode: a fresh subagent implements each task; the main session reviews between tasks. Per-task review has three stages — implementer, spec-compliance reviewer, code-quality reviewer — before the task is marked done.
- During execution: `/tidy` (light mode) sweeps the diff for dead code, duplication, and obvious bloat before commits. The `tidy` skill's deep mode targets a specific directory.

Subagent dispatch discipline (max_turns, concrete exemplars, the C3 nested-CLAUDE.md gotcha): see `subagent-dispatch.md`.

---

## Phase 4 — PR

Surfaces the change for review on the project's git host.

- Skill: `pr-and-review`.
- Host-agnostic: detects GitHub (`gh`), Gitea (`tea`), or GitLab (`glab`) from the remote URL and shows equivalent commands for each.
- Author/approver separation: where the host and policy allow, separate identities create the PR vs. approve and merge it. Reduces self-review risk.
- PR description explains *why* the change is being made; the diff already shows *what*. Test plan is a checklist.

---

## Checkpoint 2 — Code review (before merge)

**Three reviewers dispatched in parallel** against the PR diff:

| Reviewer | Mechanism | Coverage |
|---|---|---|
| Claude `code-reviewer` (sonnet) | Agent dispatch — quick pass | Security basics, style, obvious bugs, tests |
| Claude `in-depth-code-reviewer` (opus) | Agent dispatch — thorough | Architecture, test coverage, edge cases, operational impact |
| Gemini | `gemini-review.sh <PR-number>` | Cross-model coverage of single-model blind spots |

All three are dispatched in a single message (parallel tool calls). Reviewers fetch the diff via the host CLI rather than reading files locally — local files reflect master, not the PR branch, and produce false positives.

Aggregate verdicts:
- All `PASS` or `WARN` (no `BLOCK`): merge proceeds.
- Any `BLOCK`: merge is held. Findings posted; implementer addresses them; reviewers re-run on the fixed diff.

When the PR branch is behind base, the host typically blocks the merge. Recovery: rebase, force-push, **re-approve** (force-push invalidates prior approvals), then merge.

For Gemini review configuration and the `--security`/`--plan` flags, see the `pr-and-review` skill.

---

## Phase 5 — Merge & close

Lands the change and updates persistent state.

- Skill: `session-completion`.
- Order: light tidy → commit remaining state → push → PR review (if not already complete) → merge → retrospective.
- **Tasks export:** open tasks are written to `tasks.jsonl` at repo root. The next session reads this on `session-start`.
- **Memory update:** when something during the session took more than one attempt, the lesson is recorded — in auto-memory (`~/.claude/projects/<slug>/memory/`) for agent-global preferences, or in `DEVSTUFF.md` / equivalent for project-specific knowledge a human teammate benefits from.
- **failure-patterns.md update:** generalizable lessons that will recur across future sessions go into the kit's `failure-patterns.md` catalog.

---

## Persistent state across sessions

Three artifacts carry context between sessions:

| Artifact | Read | Written | Role |
|---|---|---|---|
| `tasks.jsonl` (repo root) | `session-start` restores tasks via `TaskCreate` | `session-completion` exports open tasks | Cross-session task continuity |
| `failure-patterns.md` (project root or `docs/`) | Consulted when a session is stuck or about to repeat a known mistake | New entry when a recurring, preventable mistake is found | Pattern-match catalog |
| Auto-memory (`~/.claude/projects/<slug>/memory/`) + `DEVSTUFF.md` | Auto-memory loads `MEMORY.md` every turn; `DEVSTUFF.md` is read by the developer agent on bootstrap | Updated when a durable lesson is learned | Typed memory: user / feedback / project / reference |

See `memory-system.md` for the typed-entry structure and rules about what to save vs not save.

---

## When the lifecycle does not apply

Trivial work — a typo, a comment fix, a one-character config change — bypasses this lifecycle. Boundary heuristic: if the change involves non-obvious decisions, it is in scope.

Both checkpoints are positioned to catch defects at the cheapest stage. Plan-stage defects are addressed against the plan document; code-stage defects against the diff. Catching either after merge is more expensive than catching either before merge.

---

## Quick reference

| Phase | Skill / agent | Output |
|---|---|---|
| Design | `superpowers:brainstorming` | spec doc |
| Plan | `superpowers:writing-plans` | plan doc |
| **Plan review** | `plan-reviewer` + Gemini `--plan` | verdict |
| Execute | `superpowers:subagent-driven-development` + `developer` agents + `tidy` | commits on a worktree branch |
| PR | `pr-and-review` skill | PR open on host |
| **Code review** | `code-reviewer` + `in-depth-code-reviewer` + Gemini | aggregated verdict |
| Merge & close | `session-completion` skill | merged + `tasks.jsonl` written + memory updated |

## See also

- `directive-groups.md` — the always-on contract structure
- `brainstorm-plan-execute.md` — phases 1–3 detail
- `subagent-dispatch.md` — when and how to dispatch subagents
- `worktrees-and-parallelism.md` — phase 3 isolation safety
- `verification-discipline.md` — verification standard (cross-cutting)
- `command-tiers.md` — operation tiers (cross-cutting, especially phases 4–5)
- `memory-system.md` — typed memory entries
