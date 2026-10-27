# Danimoth skills

Agent Skills for the Danimoth ETF portfolio engine (MCP server at
`https://api.danimoth.com/mcp`). Canonical source in the Danimoth monorepo:
`.agents/skills/danimoth/` — keep this export in sync with it.

## Install

```bash
npx skills add RobertoDorata/Danimoth-skills --skill danimoth -y
```

Works with Claude Code, Codex, Cursor and 40+ other coding agents via the
open `skills` CLI. Local check without publishing:

```bash
npx skills add ./open-source/danimoth-skills/danimoth -y --all
```

## Publish (maintainer)

1. Create the public GitHub repo `RobertoDorata/Danimoth-skills` (or fork under your org).
2. Push the contents of this directory as the repo root (`danimoth/` folder
   included, this README at root).
3. Verify from a clean directory: `npx skills add <owner>/skills -y --all`
   installs `danimoth` and the agent can list the MCP tools after connecting
   per `danimoth/SKILL.md`.

Onboarding: https://danimoth.com/agents ·
Machine-readable: https://api.danimoth.com/agents.md
