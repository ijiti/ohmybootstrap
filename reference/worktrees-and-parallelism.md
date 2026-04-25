# Worktrees and Parallelism

Safe patterns for running multiple agents or sessions against the same repo.

## Why worktrees

A git worktree is a second checkout of the same repo on a different branch, sharing the `.git` directory but with an independent working tree. They let you:

- Run parallel sessions without stepping on each other.
- Isolate in-progress feature work from the default branch.
- Enforce "no commits to default branch" as a structural invariant.

## Creating a worktree

```bash
git worktree add <path> -b <branch>
```

Common convention: `/home/<user>/worktrees/<repo>-<slug>/` on branch `work/<slug>`. If your project ships a wrapper (`tools/worktree-create.sh`), use it.

## The wrong-branch gotcha

**Problem:** An agent is dispatched to do work "in the worktree at `/path/to/worktree`" but `cd`s to the main repo root instead. Commits land on the default branch and policy is silently bypassed.

**Prevention:**
- Start every dispatch prompt with the working-directory invariant: "Work in `<absolute-worktree-path>`. Verify with `pwd` and `git branch --show-current` before any commit."
- After the agent reports complete, verify the commit landed on the expected branch: `git log --oneline -5` on the worktree, then check the default branch's log for accidental lands.

## The worktree-race gotcha

Two agents creating worktrees at the same moment can race on `.git/config.lock`. You'll see an error like `fatal: could not lock config file`. Retry after 1–2 seconds.

## Serializing when you must

Some things don't parallelize well:
- Agents that share a browser (Playwright MCP is single-browser-per-session; dispatching multiple Playwright-using agents in parallel causes navigate/snapshot races).
- Agents that modify the same files (git conflicts).
- Agents that contend for the same external resource (rate-limited APIs, exclusive databases).

When in doubt, run one at a time.

## Cleanup

After a branch is merged:
```bash
git worktree remove <path>
git branch -d work/<slug>
```

**Don't remove worktrees you didn't create.** Another session may be using one.
