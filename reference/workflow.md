# Workflow

Textual companion to the architecture diagram. Describes the lifecycle a non-trivial change moves through under this kit: an always-on contract, five phases, two cross-model review checkpoints, and persistent state that crosses sessions.

For trivial work (a typo fix, a one-line config bump), this lifecycle is overkill — skip it. For anything you'd write a spec or a plan for, follow it.

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

**Goal:** a written spec describing what's being built, why, and what's out of scope.

- Skill: `superpowers:brainstorming` (recommended dependency).
- Process: ask questions one at a time, propose 2–3 approaches with trade-offs, recommend one.
- Output: spec at `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` (the superpowers plugin's default location) or your project's design-doc directory.
- Self-review the spec before moving on: placeholder scan, internal consistency, scope check, ambiguity check.

Anti-pattern: "this is too simple to need a design." Even a two-sentence design surfaces assumptions that would otherwise leak into code.

See `brainstorm-plan-execute.md` for the spec-writing detail.

---

## Phase 2 — Plan

**Goal:** a step-by-step implementation plan where each step is 2–5 minutes of concrete work.

- Skill: `superpowers:writing-plans`.
- Process: TDD-shaped per task (write failing test, run to verify, implement, run to verify, commit).
- No placeholders — "add appropriate error handling" is a plan failure. Code blocks include the actual code.
- Output: plan at `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.

---

## Checkpoint 1 — Plan review (before code)

**Two reviewers dispatched in parallel** against the plan document:

| Reviewer | Mechanism | Coverage |
|---|---|---|
| Claude `plan-reviewer` (sonnet) | Agent dispatch | Scope, sequencing, risk, test plan, ambiguity |
| Gemini | `gemini-review.sh --plan <path>` | UX / frontend fit, structural clarity, single-model blind spots |

The pattern is concretely: send a single message with two tool calls — Agent tool for the Claude reviewer, Bash tool for the Gemini script — so they run concurrently.

The checkpoint runs **before any implementation work begins**. Plan-stage defects (scope creep, missing dependencies, untestable acceptance criteria, sequencing bugs) are addressed against the plan document, not against code.

Verdicts: `BLOCK` halts forward motion; `WARN` is logged and addressed if non-trivial; `PASS` proceeds.

Skip this checkpoint only for trivial chores (a `tasks.jsonl` retire, a 1-line config bump). Anything that has a plan document gets plan-reviewed.

---

## Phase 3 — Execute

**Goal:** turn the plan into code, one task at a time, with verification gates.

- Worktree-isolated: every session gets its own git worktree on a `work/<slug>` branch. Commits never land on the default branch directly. See `worktrees-and-parallelism.md` for safety patterns and the wrong-branch gotcha.
- Skill: `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans`.
- Subagent-driven mode: a fresh subagent implements each task; the main session reviews between tasks. Per-task review has three stages — implementer, spec-compliance reviewer, code-quality reviewer — before the task is marked done.
- During execution: `/tidy` (light mode) sweeps the diff for dead code, duplication, and obvious bloat before commits. See the `tidy` skill for deep-mode usage on a directory target.

Subagent dispatch discipline (max_turns, concrete exemplars, the C3 nested-CLAUDE.md gotcha): see `subagent-dispatch.md`.

---

## Phase 4 — PR

**Goal:** put the change in front of reviewers via the project's git host.

- Skill: `pr-and-review`.
- Host-agnostic: detects GitHub (`gh`), Gitea (`tea`), or GitLab (`glab`) from the remote URL and shows equivalent commands for each.
- Author/approver separation: where the host and policy allow, separate identities create the PR vs. approve and merge it. Reduces self-review risk.
- The PR description should explain *why*, not restate the diff. Test plan as a checklist.

---

## Checkpoint 2 — Code review (before merge)

**Three reviewers dispatched in parallel** against the PR diff:

| Reviewer | Mechanism | Coverage |
|---|---|---|
| Claude `code-reviewer` (sonnet) | Agent dispatch — quick pass | Security basics, style, obvious bugs, tests |
| Claude `in-depth-code-reviewer` (opus) | Agent dispatch — thorough | Architecture, test coverage, edge cases, operational impact |
| Gemini | `gemini-review.sh <PR-number>` | Cross-model coverage of single-model blind spots |

All three are dispatched in a single message (parallel tool calls). Each is told to fetch the diff via the host CLI, not read files locally — local files reflect master, not the PR branch, and produce false positives.

Aggregate verdicts:
- All `PASS` or `WARN` (no `BLOCK`): approve and merge.
- Any `BLOCK`: do not merge. Findings posted; implementer fixes; reviewers re-run on the fixed diff.

If the PR branch is behind base, the host typically blocks the merge: rebase, force-push, **re-approve** (force-push invalidates prior approvals), then merge.

For details on Gemini review configuration and the `--security`/`--plan` flags, see the `pr-and-review` skill.

---

## Phase 5 — Merge & close

**Goal:** land the change cleanly and update persistent state.

- Skill: `session-completion`.
- Order: light tidy → commit any remaining state → push → PR review (if not already done) → merge → retrospective.
- **Tasks export:** open tasks written back to `tasks.jsonl` at repo root. The next session reads this on `session-start`.
- **Memory update:** when something during the session took more than one attempt, the lesson is recorded — either in the user's auto-memory (`~/.claude/projects/<slug>/memory/`) or, if the lesson is project-specific and a human teammate would benefit, in `DEVSTUFF.md` / equivalent.
- **failure-patterns.md update:** if the lesson is generalizable (will recur in future sessions on this or other projects), it goes in the kit's `failure-patterns.md` catalog.

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

## When to skip the workflow

Trivial work — a typo, a comment fix, a one-character config change — does not benefit from this lifecycle. Use judgment: if the work involves non-obvious decisions, the workflow pays for itself.

The lifecycle exists because plan-stage defects are cheaper to fix pre-code than post-code, and code-stage defects are cheaper to fix pre-merge than post-merge. Both checkpoints exist to catch defects at the cheapest stage.

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
