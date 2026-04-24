# APPLY.md — Apply This Kit to a Target Project

**For the Claude agent reading this file:** you are being asked to adapt the target project you are currently running in (the "target repo") to resemble the operating patterns captured in this kit. Follow the numbered steps below. Each step says "propose, get approval, then write" — never write files to the target repo without showing the user your plan first.

**For humans:** tell a Claude Code session: "Read `APPLY.md` in the ohmybootstrap repo and adapt this project to match. Ask me before each change."

## Context you need

Read these before starting:
- This file (you're here).
- `README.md` — audience and scope.
- `claude/failure-patterns.md` — hard-won lessons the kit codifies.

Do not pre-load every reference doc. Read each one lazily when a step points at it.

## Step 1: Orient

Tell the user what this kit is, then survey the target project. Specifically:

- Identify the primary language/stack.
- Identify the build system (package manifest, Makefile, etc.).
- Identify the test runner.
- Identify the deployment story (if any).
- Identify the git host (`git remote get-url origin` → GitHub / Gitea / GitLab / self-hosted / unknown).
- Identify major subdirectories (ones that warrant their own nested CLAUDE.md).
- Check for existing: `CLAUDE.md`, `.claude/agents/`, `.claude/skills/`, `DEVSTUFF.md`, `MEMORY.md`, `tasks.jsonl`, `.claude/settings.json`.
- Check whether the superpowers plugin is installed (look for `~/.claude/plugins/cache/claude-plugins-official/superpowers/` or its marketplace source).

Report findings to the user as a short bulleted summary.

## Step 2: Prerequisite check

If superpowers is not installed, offer to install it or proceed without it. The kit's references to `superpowers:*` skills degrade gracefully — the CLAUDE.md template mentions them as recommended, not required. Note the user's choice.

If no git remote, ask the user how they'd like to handle host-dependent integrations (PR skills assume a remote).

## Step 3: Propose a root CLAUDE.md

Read `claude/CLAUDE.md.template`. Resolve each `<FILL IN: ...>` placeholder using what you learned in Step 1. For each non-obvious resolution, show the user your proposed text inline.

If an existing `CLAUDE.md` is present in the target, do not overwrite — show a proposed **merge**: substrate-distilled discipline on top of project-specific content that must be preserved. The user reviews the merge before you write.

Wait for approval. Then write `CLAUDE.md` at the target repo root.

## Step 4: Propose nested CLAUDE.md files (optional)

For each major subdirectory the user agrees warrants one, propose a short (one-page) nested CLAUDE.md. One at a time. Wait for approval between each.

Skip this step if the user doesn't want nested CLAUDE.md files yet — they're easy to add later.

## Step 5: Install agents

Copy `claude/agents/*.md` to `~/.claude/agents/` (user-global). If any file already exists there, **show a diff to the user** and ask before overwriting. Do not clobber local customizations silently.

Inform the user that Claude Code snapshots the agent registry at session start — the newly installed agents won't register until the session is restarted.

## Step 6: Install skills

Copy `claude/skills/*` (including the SKILL.md files and any subdirectory content) to the target repo's `.claude/skills/` (project-local) by default. If the user prefers user-global, target `~/.claude/skills/` instead. Same overwrite discipline as agents.

## Step 7: Install `failure-patterns.md` and `reference/`

Copy `claude/failure-patterns.md` to the target repo root (or to a `docs/` subdirectory the user names). Copy the entire `reference/` directory to the target repo root (preserving the directory name). The CLAUDE.md you wrote in Step 3 cross-references both — make sure the paths match the install location.

If the user has a strong preference against adding a `reference/` dir to their repo, offer two alternatives: (a) inline the relevant reference content into CLAUDE.md; (b) remove the cross-reference lines entirely. Don't silently drop the references — the CLAUDE.md template assumes they resolve.

## Step 8: Scaffold auto-memory

If `~/.claude/projects/<target-slug>/memory/` does not exist, create it and seed with `claude/MEMORY.md.template` (resolving `<FILL IN: Project Slug>`). If it exists, leave it alone — don't clobber working memory.

The `<slug>` is Claude Code's project-directory encoding — usually the repo's absolute path with `/` replaced by `-`. Ask the user for the exact slug if unsure.

## Step 9: Scaffold `tasks.jsonl`

If no `tasks.jsonl` exists at the target repo root, create an empty one.

## Step 10: Scaffold `.claude/settings.json`

If absent, propose a minimal one (an empty `permissions` block as a starting point). Skip if present.

## Step 11: Summary

Report what you wrote, what you skipped, and what changed. Remind the user:

- If superpowers was just installed, run `/mcp` (or equivalent) to reconnect the plugin.
- Restart the Claude Code session so the newly installed agents register.
- Any follow-up work the user explicitly deferred in earlier steps.

## Principles

- **Never bulk-write.** Each artifact: propose, get approval, write.
- **Never clobber.** If a file exists, show the user the diff before overwriting.
- **Prefer the target's conventions.** If the project already has an opinionated directory layout, skill location, or memory organization, adapt rather than impose.
- **The template is shape, not content.** The specific wording of S1–S4, W1–W5, etc., is substrate-distilled; what S/W/C/X slot into is project-specific. Fill in, don't blindly copy.
