---
name: tidy
description: "Code health sweep — finds duplication, dead code, and performance issues. Use with a path for deep sweep, or no args for light pre-commit check."
user_invocable: true
---

# Tidy: Code Health Sweep

Keeps the repo clean by finding duplication, bloat, and inefficiency. Two modes based on invocation.

## Mode Detection

- **Deep mode:** Invoked with a target path (e.g., `/tidy src/payments`). Full sweep with 3 parallel agents.
- **Light mode:** Invoked with no args (`/tidy`). Quick single-agent pass on git diff. Also runs automatically during session-completion.

---

## Light Mode (no args)

Run when invoked without arguments or during session-completion before commit.

**Step 1:** Get the diff.

```bash
git diff HEAD
```

If the diff is empty, check for staged changes:

```bash
git diff --cached
```

If both are empty, print "Nothing to tidy — no changes detected." and stop.

**Step 2:** Launch a single agent (type: `code-simplifier:code-simplifier`, max_turns: 15) with this prompt:

> You are reviewing a git diff for code health issues. Here is the diff:
>
> &lt;paste diff&gt;
>
> Check for these issues ONLY (be fast, not exhaustive):
>
> 1. **Duplication** — Do any new functions or types closely match something already in the repo? Use Grep to search for similar function names and patterns in the project's shared-code directories (consult CLAUDE.md for the list, or fall back to grep-wide).
> 2. **Dead code** — Are there new functions nothing calls? Did removing code leave orphaned functions elsewhere?
> 3. **Obvious bloat** — Unused imports, unreachable branches, unnecessary nil checks in new code.
>
> **Auto-fix:** Remove dead imports and unused variables. Nothing else.
>
> **Report format:**
> ```
> ## Tidy Report (light)
>
> ### Findings
> - **TAG** file:line — description. Recommendation.
>
> ### Auto-fixed
> - List of fixes applied
>
> ### Summary
> Clean / N issues found
> ```
>
> If everything is clean, just print "Clean." and stop.

**Step 3:** If the agent reports structural issues (DUPLICATE or CANDIDATE tags), ask the user:

"Tidy found N issue(s). Address before committing, or proceed?"

If light mode was invoked manually, just print the report.

---

## Deep Mode (with target path)

Run when invoked with a path argument.

**Step 1:** Validate the target path exists.

```bash
ls <target_path>
```

If it doesn't exist, print "Path not found: &lt;target_path&gt;" and stop.

**Step 2:** Read the target's CLAUDE.md if one exists (for project context, language, and conventions).

**Step 3:** Launch 3 agents in parallel using the Agent tool. All three should be launched in a single message. Each agent gets the target path and instruction to search the full repo for cross-references.

### Agent 1: Consolidation (type: `code-reviewer`, max_turns: 30)

> You are auditing `<target_path>` for consolidation opportunities.
>
> **Your priorities (in order):**
>
> 1. **Cross-project duplication** — Search the project's shared-code directories (consult the root CLAUDE.md for the list, or fall back to grep-wide) for functions that do the same thing as functions in the target. If two modules implement the same retry loop, HTTP client wrapper, scoring function, or data transformation — flag it.
>
> 2. **Internal duplication** — Within the target path, find copy-paste blocks or near-identical functions that should be unified into a shared helper.
>
> 3. **Missed reuse** — Check if the target reimplements anything already provided by the language's standard library or shared packages. Common culprits: custom error types, retry logic, HTTP helpers, string parsing, file I/O wrappers.
>
> 4. **Structural consolidation** — Small files that should merge, modules that overlap in purpose, types defined in multiple places.
>
> **DO NOT make changes.** Report only.
>
> **Report format:**
> ```
> ### Consolidation
> - **DUPLICATE** file:line — description. Recommend: use <existing_function> from <package> instead.
> - **CANDIDATE** file:line and file:line — description. Recommend: extract to shared helper.
> - **REUSE** file:line — reimplements <stdlib/shared function>. Recommend: use <function> directly.
> ```

### Agent 2: Bloat & Dead Code (type: `code-simplifier`, max_turns: 30)

> You are auditing `<target_path>` for bloat and dead code.
>
> **What to find:**
>
> 1. **Unused exports** — Public functions, types, or constants with no callers outside their module. Use Grep across the full repo to verify.
>
> 2. **Unreachable code** — Branches after unconditional returns, impossible type assertions, error checks on infallible operations.
>
> 3. **Stale files** — Source files not imported or referenced by anything. Check with Grep for import paths.
>
> 4. **Over-engineering** — Interfaces with one implementor, abstractions with one user, config for values that never change, generic helpers called from one place.
>
> 5. **Comment rot** — TODOs for completed work, comments describing deleted behavior, commented-out code blocks.
>
> **Auto-fix these:** Dead imports, unused unexported functions/variables (verify with Grep first), commented-out code blocks.
>
> **Report everything else.**
>
> **Report format:**
> ```
> ### Bloat
> - **REMOVED** file:line — description (auto-fixed).
> - **DEAD** file:line — description. Confirm removal.
> - **OVERENG** file:line — description. Recommend: simplify to <alternative>.
> - **STALE** file — no importers found. Confirm deletion.
> ```

### Agent 3: Performance & Efficiency (type: `backend-generalist`, max_turns: 25)

> You are auditing `<target_path>` for performance and efficiency issues.
>
> Adapt the hot-path heuristics to the project's language (e.g., N+1 patterns, unnecessary allocations, missed concurrency, resource leaks). Read CLAUDE.md for language and conventions.
>
> **What to find:**
>
> 1. **N+1 patterns** — Database queries or API calls inside loops. Should be batched.
>
> 2. **Missed concurrency** — Sequential independent operations (multiple HTTP calls, multiple file reads, multiple DB queries for different data) that could run in parallel.
>
> 3. **Unnecessary allocations** — Repeated object creation in hot paths, growing collections without size hints, repeated string concatenation in loops.
>
> 4. **Hot-path bloat** — Expensive operations (file I/O, JSON marshal/unmarshal, reflection, regex compilation) inside request handlers or tight loops that could be hoisted.
>
> 5. **Resource leaks** — Opened files/connections/responses without cleanup (language-appropriate: `defer`, `finally`, `with`, `using`, etc.).
>
> **DO NOT make changes.** Report only. Performance changes always need human judgment.
>
> **Report format:**
> ```
> ### Performance
> - **N+1** file:line — description. Recommend: batch query.
> - **CONCURRENCY** file:line — description. Recommend: parallelize.
> - **ALLOC** file:line — description. Recommend: pre-allocate or reuse.
> - **HOTPATH** file:line — description. Recommend: hoist to init/setup.
> - **LEAK** file:line — description. Recommend: add cleanup.
> ```

**Step 4:** Wait for all 3 agents to complete. Aggregate their reports into a single tidy report:

```
## Tidy Report: <target_path>

<Agent 1 consolidation section>

<Agent 2 bloat section>

<Agent 3 performance section>

### Auto-fixed
<combined list from agents that made changes>

### Summary
- N consolidation opportunities (manual)
- N bloat/dead code findings (N auto-fixed, N need confirmation)
- N performance issues (manual)
```

**Step 5:** If any agents auto-fixed code, run the project's test command to verify nothing broke:

```bash
# Run the project's test command. Check CLAUDE.md for the exact invocation.
# Common defaults:
#   Go:       go test ./...
#   Node:     npm test  or  pnpm test
#   Python:   pytest
#   Rust:     cargo test
```

If tests fail, revert the auto-fixes and report the failure.
