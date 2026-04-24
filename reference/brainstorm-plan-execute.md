# Brainstorm → Plan → Execute

A workflow for non-trivial work. The superpowers plugin (recommended) provides skills for each phase: `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:executing-plans` (or `superpowers:subagent-driven-development` for dispatching-fresh-subagent-per-task execution).

## Why a three-phase workflow

Agents — like humans — are worst at holding a whole design in their head while typing code. Mistakes at the typing level are cheap; mistakes at the design level are expensive and often not discovered until much later. Separating phases:

- **Brainstorm** forces the design to be explicit before code is written.
- **Plan** turns the design into concrete, ordered, testable steps.
- **Execute** turns each step into code, one at a time, with verification gates.

## Phase 1: Brainstorming

Goal: a written spec that describes what's being built, why, and what's out of scope.

- Ask questions one at a time to refine the idea.
- Propose 2–3 approaches with trade-offs; recommend one.
- Write the spec to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`.
- Self-review the spec: placeholder scan, internal consistency, scope check, ambiguity check.
- User reviews the written spec before moving to phase 2.

**Anti-pattern:** "this is too simple to need a design." Even small tasks benefit from a two-sentence design — it surfaces assumptions.

## Phase 2: Planning

Goal: a step-by-step implementation plan where each step is 2–5 minutes of concrete work.

- Each task: file paths, exact code, exact commands, expected output.
- TDD-shaped where feasible (write failing test, run to verify, implement, run to verify, commit).
- No placeholders. "Add appropriate error handling" is a plan failure.
- Optional but recommended: dispatch `plan-reviewer` (and a cross-model reviewer if available) against the plan before starting implementation. Plan defects are 10× cheaper to fix pre-code.

Save to `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`.

## Phase 3: Execution

Two execution modes:

- **Inline execution** — work through the plan in the current session, committing after each task.
- **Subagent-driven execution** — dispatch a fresh subagent per task, review between tasks. Faster iteration, more token cost, better isolation.

Each task's verification step must actually run before the task is marked complete.

## When to skip the workflow

Trivial chores — a one-line config bump, renaming a variable, fixing a typo — don't need a spec or a plan. Use judgment: if the work involves non-obvious decisions, the workflow pays for itself.
