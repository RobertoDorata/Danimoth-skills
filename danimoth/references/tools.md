# Danimoth MCP tools — reference

Endpoint: `https://api.danimoth.com/mcp` (Streamable HTTP, stateless).
All percentages are fractions (`0.12` = 12%); money is EUR.

| Tool | Mutating | Auth | Backend route | Notes |
|---|---|---|---|---|
| `dashboard_metadata` | no | token in prod | `GET /api/meta` | Classes, horizons, confidences, defaults, data window. Call first. |
| `search_etfs` | no | token in prod | `GET /api/catalog` | `search`, `criteria` (plain language), `offset`, `limit` (default 50). |
| `profiling_questionnaire` | no | token in prod | `GET /api/questionnaire` | MiFID-style questionnaire + answer schema. |
| `list_clients` | no | required | `GET /api/clients` | Advisor-scoped; never share across advisors. |
| `save_client` | yes | required | `POST /api/clients` | Only after explicit advisor confirmation. `client_id` + Acc/Dist preference. |
| `propose_portfolio` | no | required with `client_id` | `POST /api/portfolio` | Weights, estimators, scenarios, tax. No orders, ever. |

Key `propose_portfolio` inputs: `cvar_limit`/`var_limit`, `capital`,
`horizon_days` (252 = 1y), `alpha` (0.975), `max_ter`, `distribution_policy`
(`Accumulating`/`Distributing`), `client_id`, `allowed_isins`,
`target_weights` (explicit, never normalised), `tax_loss_balances`,
`current_positions`, `observed_path`.

Reading results:

- Risk is quoted as positive losses ("CVaR 12%" = mean loss 12% in the worst
  2.5% of 1-year scenarios). Block bootstrap (63 sessions) is the default;
  historical overlapping windows underestimate — show all estimators.
- `infeasible` keeps the proposal visible with unchanged weights; explain
  which constraint binds instead of hiding it.
- `snapshot_id` ties comparisons to one proposal; do not mix comparisons
  across snapshots.
