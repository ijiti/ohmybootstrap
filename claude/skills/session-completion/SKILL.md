---
name: session-completion
description: "Use when ending a session or completing work — handles commit, push, PR creation, review dispatch, task export, and session retrospective."
---

# Session Completion

Follow this checklist when ending a session or when the user says "we're done." Never leave a session without completing these steps.

## Checklist

1. **Run tidy (light mode)** — Invoke `/tidy` with no args. If structural issues are reported, ask whether to address them before committing.

2. **Commit all work** — `git add <specific-files>` (never `git add -A` or `git add .`). Write clear commit messages that explain *why*.

3. **Push the branch** — `git push -u origin <branch-name>`.

4. **Create a PR** — Use the project's host tool:
   - GitHub: `gh pr create --title "..." --body "..." --base <default> --head <branch>`
   - Gitea: `tea pr create --repo <owner/repo> --title "..." --description "..." --base <default> --head <branch>`
   - GitLab: `glab mr create --title "..." --description "..." --target-branch <default> --source-branch <branch>`

5. **Dispatch code review** — Invoke the `pr-and-review` skill.

6. **Export open tasks** — Write all open tasks to `tasks.jsonl` at the repo root (one JSON object per line). Commit to the branch and include in the PR.

## Always

- **Push must succeed.** Never end a session with unpushed work.
- **Export tasks to `tasks.jsonl`.** Format: one JSON object per line with fields: `subject`, `description`, `status`, `owner`, `blockedBy`.
- **Session retrospective:** Ask yourself — *what did I struggle with this session?* If anything took more than one attempt:
  - Update auto-memory (`~/.claude/projects/<slug>/memory/`).
  - Fix incomplete docs or add missing examples.
  - The next session should never hit the same wall.
