---
name: pr-and-review
description: "Use when creating PRs, dispatching code reviews, or merging — host-agnostic workflow with Claude + optional cross-model review."
user_invocable: true
---

# PR and Code Review

Complete workflow for creating PRs, dispatching reviewers, posting results, and merging. Reviewers only review — the main session handles host interaction, verdict decisions, and merging.

## Host Detection

Check which git host the repo uses:

```bash
git remote get-url origin
```

- `github.com/...` → use `gh`
- `gitea.<whatever>/...` → use `tea`
- `gitlab.<whatever>/...` → use `glab`
- Self-hosted / unknown → ask the user.

Each host's CLI has equivalent verbs for: `pr create`, `pr view/diff`, `pr approve`, `pr merge`, `comment`. Subsections below show the exact command per host.

## Creating a PR

First, commit your work and push the branch:

```bash
git push -u origin <branch-name>
```

Then create the PR with your host's CLI:

**GitHub:**
```bash
gh pr create --title "feat: short description" \
  --body "What changed and why" \
  --base <default-branch> --head <branch-name>
```

**Gitea:**
```bash
tea pr create --repo <owner/repo> \
  --title "feat: short description" \
  --description "What changed and why" \
  --base <default-branch> --head <branch-name>
```

**GitLab:**
```bash
glab mr create --title "feat: short description" \
  --description "What changed and why" \
  --target-branch <default-branch> --source-branch <branch-name>
```

**Branch naming:** lowercase alphanumeric with hyphens (e.g. `fix-login-timeout`, `feat-dark-mode`).

**Git policy:**
- NEVER commit directly to the default branch. All changes — no matter how small — go through branch → PR.
- If your host and policy allow, separate the PR-author and PR-approver identities. On GitHub this usually means a second user or a bot account with write access; on Gitea, separate logins in `~/.config/tea/config.yml`.
- **Emergency escape hatch:** For critical security fixes that cannot wait for review, some hosts allow an admin override to merge without approval. Still create a branch and PR — just skip waiting for review. Document the reason in the PR description.

## Choosing Reviewers

| PR Type | Claude Reviewer | Cross-model |
|---------|----------------|-------------|
| Small/routine | 1x `code-reviewer` (sonnet) | Optional Gemini |
| Large | Multiple `code-reviewer` by file group | Optional Gemini |
| Critical/security | `in-depth-code-reviewer` (opus) | `--security` flag |

**Cross-model review is strongly recommended.** It's fast (~20s), catches model-specific blind spots, and provides cross-model consensus.

## Dispatching Reviews

**Dispatch all reviewers in parallel** — single message, multiple tool calls:

```
# Claude reviewer via Agent tool:
Agent tool → subagent_type: "code-reviewer" (or "in-depth-code-reviewer")
Prompt: "Review PR #<N>. Fetch the diff with:
  <host-cli> pr diff <N>   (or equivalent — see PR View/Diff section)
Return your structured verdict."

# Cross-model reviewer via Bash tool (same message, if available):
Bash → tools/gemini-review.sh <N>   (or equivalent wrapper)
```

Send both tool calls in the same message so they run concurrently.

### Review Prompt Templates

Always tell reviewers to use the host CLI to fetch the PR diff — never paste diffs inline or let reviewers read files from the working tree. Reviewers reading local files will see the default-branch version, not the PR branch, and produce false positives.

**Quick review:**
```
Review PR #<N>. Fetch the diff with:
  <host-cli> pr diff <N>
Base your review ONLY on the diff content — do NOT read files from the local
filesystem as they may not reflect the PR branch. Return your structured
verdict.
```

**Thorough review:**
```
Review PR #<N>. Fetch diff and details with:
  <host-cli> pr diff <N>
Read CLAUDE.md for project conventions and security policy. Base your review
ONLY on the diff and PR metadata — do NOT read changed files from the local
filesystem as they reflect the default branch, not the PR branch. Return your
structured findings.
```

**File review (no PR):**
```
Review these files for issues: <paths>. Focus on changes vs the default branch.
```

## PR View / Diff

**GitHub:**
```bash
gh pr diff <N>
gh pr view <N>
```

**Gitea:**
```bash
tea pr --repo <owner/repo> --output plain <N>
```

**GitLab:**
```bash
glab mr diff <N>
glab mr view <N>
```

## Posting Results

Post review comments on the PR using your host's CLI:

**GitHub:**
```bash
gh pr review <N> --comment --body "Review findings..."
```

**Gitea:**
```bash
tea comment --repo <owner/repo> <N> "Review findings..."
```

**GitLab:**
```bash
glab mr note <N> --message "Review findings..."
```

Post each reviewer's output as a separate comment, then post your verdict comment synthesizing all findings.

The verdict comment is the merge gate.

## Merge Decision

- **All PASS or WARN (no BLOCKs):** Approve and merge.
- **Any BLOCK:** Do not merge. Post findings, note what needs fixing.

**GitHub:**
```bash
gh pr review <N> --approve --body "LGTM"
gh pr merge <N> --rebase
```

**Gitea:**
```bash
tea pr approve --repo <owner/repo> <N> "LGTM"
tea pr merge --repo <owner/repo> --style rebase <N>
```

**GitLab:**
```bash
glab mr approve <N>
glab mr merge <N> --rebase
```

### Rebase-before-merge

Branch protection rejects merges when the head branch is behind the base branch. When you hit this:

```bash
# In the feature branch
git fetch origin <default-branch>
git rebase origin/<default-branch>
git push --force-with-lease origin <branch-name>
```

**After any force-push, the previous review approval may be invalidated by the SHA change.** Check your host's policy. If required, re-approve before merging.

**Verify the merge actually landed.** Some CLIs return misleading error messages even when the merge succeeded server-side. Confirm by fetching the PR state via the CLI or API.

## Mid-Development Review (No PR)

Dispatch a `code-reviewer` with file paths. No host interaction needed — just get the verdict and use the feedback.

## Plan Review (Before Code)

For non-trivial work with a plan doc, run cross-model plan review **before** implementation. Plan defects are 10× cheaper to fix pre-code than post-code.

**Dispatch in parallel** — single message, two tool calls:

```
# Claude reviewer via Agent tool:
Agent tool → subagent_type: "plan-reviewer"
Prompt: "Review the plan at <path>. Return your structured verdict."

# Cross-model reviewer via Bash tool (same message, if available):
Bash → tools/gemini-review.sh --plan <path>   (common convention)
```

**Why both:** Different models prioritize differently — one may catch scope gaps while another catches sequencing bugs. Cross-model review improves coverage on UX/frontend concerns, structural clarity, and missing risks.

**When to skip:** trivial chores (a 1-line config bump, a rename) don't need a plan and therefore don't need plan review. Plan review is for anything you'd write a plan doc for.

## Cross-Model Review Details

If you have a Gemini CLI wrapper (common convention: `tools/gemini-review.sh`), dispatch it alongside the Claude reviewer. If not, use a second Claude reviewer on a different model for cross-model coverage. Cross-model review is optional — the skill degrades gracefully to Claude-only review.

```bash
# Standard review (if wrapper exists)
tools/gemini-review.sh <pr-number>

# Security-focused review
tools/gemini-review.sh --security <pr-number>

# Review a diff file directly
tools/gemini-review.sh --diff /tmp/changes.diff

# Plan review (before code)
tools/gemini-review.sh --plan docs/plans/2026-04-21-feature.md
```

## Host CLI Quick Reference

**GitHub (`gh`):**
```bash
gh pr list                                         # list open PRs
gh pr create --title "..." --body "..." \
  --base <default> --head <branch>                 # create PR
gh pr diff <N>                                     # view diff
gh pr review <N> --approve --body "LGTM"           # approve
gh pr merge <N> --rebase                           # merge (rebase)
gh pr review <N> --comment --body "..."            # post comment
gh pr close <N>                                    # close without merging
```

**Gitea (`tea`):**
```bash
tea pr ls --repo <owner/repo>                      # list open PRs
tea pr create --repo <owner/repo> \
  --title "..." --description "..." \
  --base <default> --head <branch>                 # create PR
tea pr --repo <owner/repo> <N>                     # view PR
tea pr approve --repo <owner/repo> <N> "LGTM"     # approve
tea pr merge --repo <owner/repo> \
  --style rebase <N>                               # merge (rebase)
tea comment --repo <owner/repo> <N> "..."          # post comment
tea pr close --repo <owner/repo> <N>               # close without merging
```

**GitLab (`glab`):**
```bash
glab mr list                                       # list open MRs
glab mr create --title "..." --description "..." \
  --target-branch <default> --source-branch <branch>  # create MR
glab mr diff <N>                                   # view diff
glab mr approve <N>                                # approve
glab mr merge <N> --rebase                         # merge (rebase)
glab mr note <N> --message "..."                   # post comment
glab mr close <N>                                  # close without merging
```

## Notes

- **Complex markdown bodies** (backticks, code fences, tables) can silently fail with some CLIs when passed inline — the command returns exit 0 and posts nothing. Write the body to a file (`/tmp/pr-body.md`), then pass via the CLI's file-input flag or stdin. Always verify the comment landed by re-fetching it.
- **`tea pr review` is INTERACTIVE only.** For non-interactive review-with-comment use `tea pr approve` with positional body, or post a separate `tea comment`.
