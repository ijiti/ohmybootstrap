# Failure Patterns

Specific mistakes agents have made repeatedly. Pattern-match against these; don't repeat.

## Async `await` is not a thread escape (JS/TS)

Performance work where `await` was treated as "frees the main thread" produced worse-performing code. Microtasks resolve on the main thread and carry browser-internal sync work (data copies, ImageBitmap finalization, XHR serialization). **Web Worker is the only true escape from a CPU-bound bottleneck.** Don't claim `await` fixed a jank problem without a profile.

## Subagent commits on wrong branch

Agent dispatched to a worktree but `cd`'d to the main repo root — commits landed on the default branch instead of the intended branch. **Always verify commits land on the expected branch before reporting complete.** A subagent that reports "committed and pushed" but used the wrong working directory has silently bypassed your branch policy. When parallel subagents create worktrees simultaneously, they can also race on `.git/config.lock` — retry after a few seconds if you see a lock error.

## Heredoc silently dropped complex markdown

`<tool> comment --body "$(cat <<EOF ...)"` with backticks or code fences can exit 0 and post nothing. Rule: write the body to a file (`/tmp/pr-body.md`), then `BODY=$(cat /tmp/pr-body.md) && <tool> comment ... "$BODY"`, and always verify the comment landed by re-fetching it. Some CLIs (notably `tea comment`) additionally hang on long positional-arg bodies; use stdin (`--body-file -` or `-F body=@-`) instead.

## Synthetic events don't exercise real OS input paths

Drag-drop, file-chooser, clipboard, touch tested via JS-dispatched events (`dispatchEvent(new DragEvent(...))`, `element.click()`) is **not verification** — these bypass OS-level input pipeline parts that real events trigger. Use platform automation (CDP `Input.dispatchTouchEvent`, `xdotool`, Playwright's real input methods) or mark the interaction verified-by-smoke-test-only and ask the user to confirm.

## Version-bump left contradictory prose in the same file

When updating a prompt, rubric, spec, or any doc for a new version, ALL same-file blocks (decision trees, pseudocode, status sections, bootstrap subsections, output-format JSON) must be updated together. An agent reading top-to-bottom encounters earlier-version rules first and applies them. **Grep every pseudocode fence and every decision-tree verb ("If X then Y") before committing.**

## Agent inherited `ANTHROPIC_API_KEY` and hit paid API

Dispatched subprocesses inherit env by default, so `claude -p` subprocesses run on the paid API when `ANTHROPIC_API_KEY` is set in the parent. Fix: strip the API key from the child environment in your dispatcher. Any `claude -p` subprocess must run on subscription, not paid API.

## `git commit --amend` silently aborted in compound commands

`git commit --amend -m "..." && git push --force-with-lease` can be entirely denied by a hook that flags the push, cancelling the amend too. You think the amend landed, you push later, and the amended content never shipped. **Rule:** split `commit --amend` into its own command, verify with `git show --stat HEAD`, then push separately.

## Rebasing a migration branch reintroduces what you migrated

If the default branch adds new code (from an unrelated merge) that uses the pattern you were removing, a clean rebase replays it verbatim. After any rebase of a migration/refactor branch, re-run the migration-validation grep on the full file set before pushing.

## Adding to instructions

When you hit a recurring, preventable mistake, add it here. The entry should name:
- What you tried.
- Why it looked correct.
- What actually happened.
- The rule that prevents it next time.

If the mistake is project-specific (not generalizable), put it in the project's own CLAUDE.md or a memory topic file instead of this catalog.
