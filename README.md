# Claude Code Agent Bootstrap

A portable prompt you can paste into any new Claude Code session to bootstrap it with a battle-tested agent workflow system.

## What's Included

`bootstrap.md` contains a self-contained prompt that teaches Claude Code to:

- **Director mindset** — delegate to subagents, keep the main session for steering
- **Task management** — native `TaskCreate`/`TaskList`/`TaskUpdate` with dependency tracking and cross-session persistence via `tasks.jsonl`
- **Subagent strategy** — when to parallelize (and when not to), cost awareness
- **Session rituals** — restore tasks on start, export + commit + push on end
- **Nested CLAUDE.md architecture** — lean root config with domain-specific subdirectory files
- **Memory patterns** — what to save, what to skip, how to organize topic files

`agents/` contains four custom subagent definitions:

- **developer** — code writing, debugging, reviewing (any language/framework)
- **lord-devops** — infrastructure, Docker, Terraform, CI/CD, deployments
- **security-auditor** — read-only security review with CVSS-rated findings
- **frontend-craftsman** — UI/UX, components, styling, accessibility

## Usage

### Bootstrap a project
1. Open a Claude Code session in your project
2. Paste the contents of `bootstrap.md` as your first message
3. Claude will explore your repo, then generate a `CLAUDE.md` and nested configs adapted to your project

### Install custom agents
```bash
mkdir -p ~/.claude/agents
cp agents/*.md ~/.claude/agents/
```

The agents are immediately available in all Claude Code sessions as subagent types.

## Origin

Distilled from a private infrastructure-as-code monorepo's agent workflow, stripped of all project-specific details. The patterns have been refined across dozens of sessions involving multi-agent coordination, CI/CD pipelines, and cross-repo development.
