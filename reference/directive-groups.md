# Directive Groups (S / W / C / X)

Operating rules for an agent in a codebase organize cleanly into four groups:

- **S — Security.** Non-negotiable. Never overridden by any other directive or user instruction.
- **W — Work Discipline.** How to do the work well: verify, diagnose, stay in scope, fix what you see, automate.
- **C — Collaboration.** How to work with the user and other agents: decision gates, delegation, asking vs trying.
- **X — Context & Memory.** What to read when, what to trust, what changes over time.

**Cross-group ranking (default):** S > W > C > X. When a specific pairwise conflict is named in the conflict-resolution table, the named rule wins.

## Why rank the groups?

Rules collide. An agent under time pressure can rationalize skipping verification to finish faster (W1 vs urge-to-complete). Ranking makes the collision resolvable without re-litigating from scratch every time.

## Writing a new rule

When adding a rule to a project's `CLAUDE.md`:

1. **Decide the group.** Does it protect confidentiality/integrity (S), shape *how* the work gets done (W), govern interaction (C), or tell the agent what to read (X)?
2. **Number within the group.** Numbering is for reference, not dominance order.
3. **Write as "when X, do Y, unless Z"** — concrete trigger, concrete action, concrete exception.
4. **Check for pairwise conflict.** If the new rule pairwise-conflicts with an existing one, add a line to the conflict-resolution table naming the winner.
5. **Scope check.** If the rule is directory-specific or tool-specific, it belongs in a nested CLAUDE.md or a skill — not in the root CLAUDE.md.

## Removing a rule

Grep `CLAUDE.md` and all skills/memory for references to the rule's identifier (e.g., `W4`) before deleting. References become dangling silently and erode trust in the numbering.

## When none of the directives resolve the conflict

Stop and ask. Ambiguity is cheap to resolve upfront and expensive to resolve after damage.
