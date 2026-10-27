---
name: danimoth
description: >
  Build, analyse and explain UCITS ETF portfolios with the Danimoth MCP server
  (VaR/CVaR risk budgets, inverse-volatility weights, MiFID-style profiling,
  Italian tax notes). Use when a financial advisor or their agent needs to
  search ETFs, profile a client, or propose a portfolio via Danimoth tools.
  Never submits orders.
---

# Danimoth — ETF portfolios via MCP

Use this skill when working with the Danimoth portfolio engine through its MCP
server. You are assisting an independent financial advisor; the advisor reviews
and decides. You never submit orders, never move money, and never present a
proposal as personalised advice without a saved client profile.

## 1 — Connect

Remote (preferred), Streamable HTTP:

```json
{
  "mcpServers": {
    "danimoth": {
      "url": "https://api.danimoth.com/mcp",
      "headers": { "Authorization": "Bearer <FIREBASE_ID_TOKEN>" }
    }
  }
}
```

Claude Code one-liner:

```bash
claude mcp add --transport http danimoth https://api.danimoth.com/mcp --header "Authorization: Bearer <FIREBASE_ID_TOKEN>"
```

Local stdio fallback (repo checkout): `uv run --extra mcp cfi-mcp` with
`CFI_API_URL` (default `http://127.0.0.1:8080`) and `CFI_API_TOKEN`.

Auth rules:

- The token is the advisor's Firebase ID token (sign in on danimoth.com). It
  belongs in the MCP client header or `CFI_API_TOKEN`, **never** in a tool
  argument, prompt, file or log.
- HTTP 401 from a tool means missing/expired token: ask the advisor to sign in
  again and refresh the header. Do not retry with invented credentials.
- If no MCP server is configured, fetch `https://api.danimoth.com/agents.md`
  and follow it; that page is the canonical onboarding.

## 2 — Discover before calling

List the server's tools first and read their schemas. Expected tools:
`dashboard_metadata`, `search_etfs`, `profiling_questionnaire`,
`list_clients`, `save_client` (only mutating tool), `propose_portfolio`.
Start read-only sessions with `dashboard_metadata` to learn asset classes,
limits, defaults and the catalogue date, and quote that date in your answer.

## 3 — Workflows

**Explore ETFs.** `search_etfs` with plain criteria (e.g. "physical replication,
large AUM"), paginate with `offset`/`limit`. Report TER, replication, AUM and
distribution policy when present; never invent missing fields.

**Profile a client.** Fetch `profiling_questionnaire` before asking questions.
Only call `save_client` after the advisor confirms the exact profile to store;
read back with `list_clients` and summarise what changed.

**Propose a portfolio.** `propose_portfolio` with fractions, not percents
(`cvar_limit: 0.12` = 12%), capital in EUR. With `client_id` the saved
Acc/Dist preference applies and auth is mandatory. Explain: weights per ETF,
all risk estimators shown by the engine (their divergence is information, not
an error), scenario/risk cone, benchmark/crisis comparisons if returned, and
tax notes (OICR/ETF proceeds are capital income in Italy; unverified tax
categories are never treated as compensable). State clearly: proposal only,
no order was submitted.

## 4 — Failure behaviour

- `infeasible` / HTTP 422: relay the budget problem, suggest the smallest
  constraint change (higher CVaR budget, wider TER cap, broader universe), and
  ask before recomputing. Never silently relax constraints.
- Explicit `target_weights` are never normalised or "fixed" by the engine; if
  risk exceeds the budget the proposal stays visible as infeasible. Mirror
  that: show weights unchanged.
- Backend errors are redacted by design. If a tool fails, report the HTTP
  status class and the next step; do not speculate about internals.

## 5 — References

- `references/tools.md` — tool-by-tool parameters and reading of results.
- Onboarding and discovery: `https://api.danimoth.com/agents.md`,
  `https://api.danimoth.com/llms.txt`, `https://api.danimoth.com/.well-known/mcp.json`.
