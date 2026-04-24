# Command Tiers

Operations fall into four tiers by blast radius and reversibility. Your project's CLAUDE.md should tell the agent which concrete commands live in which tier; this doc explains the tier framework.

## Tier 1 — Always allowed

Local, reversible, no shared state. Agents run these freely.
- **Examples:** reading files, running tests, typechecking, building, linting, git status, git diff, git log, listing files, searching code.
- **Rule:** no confirmation needed.

## Tier 2 — Announce and proceed

Local but visible side effects or non-trivial to revert. Agents say what they're about to do, then do it.
- **Examples:** creating a branch, creating a commit, writing a new file in-scope, running a dev server, creating a local database, git push to a feature branch.
- **Rule:** narrate the action ("I'm going to create branch `work/alpha` now"), then proceed without asking for explicit approval.

## Tier 3 — Requires authorization

Hard to reverse, visible beyond the local session, or affecting shared state. Ask before proceeding.
- **Examples:** force-push, `git reset --hard`, deleting files/branches, merging a PR, commenting on a PR, opening an issue, deploying to staging or production, rotating a credential, running a cleanup script, modifying CI/CD pipelines.
- **Rule:** describe the action, its blast radius, and any rollback path. Wait for explicit approval. Approval is scope-specific — a yes on one action is not a yes on similar actions.

## Tier 4 — Hard-blocked, no exceptions

Would cause irreversible damage or bypass safety rails.
- **Examples:** committing secrets, force-pushing to the default/protected branch, `--no-verify` to skip hooks, `--no-gpg-sign` when commits are required to be signed, disabling security tooling.
- **Rule:** refuse. If the user insists, explain why the block exists and help them accomplish the underlying goal without triggering it.

## Writing your project's tier list

In your CLAUDE.md (or a nested CLAUDE.md for a specific subsystem), explicitly enumerate which of your project's commands sit in each tier. This is how the generic framework becomes actionable.

Example stub:

```markdown
## Command Tiers (this project)

**Tier 3 — ask first:**
- `npm publish`
- `terraform apply` (any environment)
- Deleting rows from `production.users`
- Merging a PR into `main`

**Tier 4 — hard-blocked:**
- Direct `git push` to `main`
- Committing files matching `.env*`, `*.key`, `*.pem`
- `chmod 777` on anything
- `curl | sh` piping
```
