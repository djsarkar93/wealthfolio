# Portfolio Insights - Data Requirements & Feasibility

A breakdown of the Portfolio Insights feature (`/insights`) by the underlying
data each visual piece needs, independent of this codebase's own schema. Use
this as a readiness checklist when assessing whether a different holdings
dataset could reproduce the same views.

Source for the feature's own implementation: see
[feature-source-map.md](feature-source-map.md) ("Portfolio insights" section)
and [api-feature-map.md](api-feature-map.md) ("§6 Portfolio Insights") for the
exact files and calls.

## Widget-by-widget requirements

| Widget | What it shows | Minimum data needed | Tier |
| --- | --- | --- | --- |
| Asset class breakdown (donut) | share of value by asset class | a position-level table with `market_value` and an `asset_class` label per holding | Core |
| Sector breakdown (donut) | share of value by sector | same table, plus a `sector` label per holding | Core |
| Concentration card | largest holding %, top-5 % | same table, plus a stable per-holding identifier and `weight`/`market_value` | Core |
| Value strip | total value, invested, book cost, P&L | `market_value`, `cost_basis` (or an unrealized P&L figure) per holding | Core |
| Region breakdown (donut) | share of value by geography | a `region` (or country) label per holding - not implied by a plain holdings table; must be sourced or inferred per security | Needs extra attribute |
| Currency breakdown | share of value by currency | a `currency` label per holding | Needs extra attribute |
| Risk breakdown | share of value by risk tier | a per-holding risk classification - distinct from a portfolio-level risk profile, which is not granular enough | Needs extra attribute |
| Security-type breakdown | share of value by instrument type (equity, fund, etc.) | a `holding_type`/instrument-type label per holding | Needs extra attribute (often already present) |
| Composition treemap | size by value, color by day's move | `market_value` for size; a daily price-change figure for color - the latter requires either a live quote feed or at least two time-stamped snapshots per holding to compute a delta | Needs time-series data |
| Top movers | ranked gainers/losers | same daily price-change figure as above, at the holding level | Needs time-series data |
| Accounts chart | value split by account | a multi-account structure (one holding set per named account) - a single flat portfolio per client/investor cannot reproduce this without an artificial re-bucketing of holdings | Needs additional entity |
| Target allocation / drift bands | current vs. target weight per asset class, buy/sell suggestions | a persisted target-weight record per asset class (per investor or per a shared risk-based template), compared against current weights | Needs additional entity |
| Custom groups | user-defined groupings of holdings | a full taxonomy/category management system (create/assign/reassign categories per holding) - this is a feature in its own right, not a data gap | Not a data gap - separate feature |
| Fund look-through | true exposure inside an ETF/fund, by its underlying constituents | constituent-level holdings data for every fund/ETF held, refreshed on a schedule - not derivable from the fund's own ticker/position row | Needs external data source |

## Reading the tiers

- **Core** - reproducible from a plain, single-snapshot holdings table (one
  row per position, with value, asset class, and sector).
- **Needs extra attribute** - reproducible by adding a column or two to the
  same holdings table; no new entities or relationships required.
- **Needs time-series data** - requires either live market data or multiple
  historical snapshots per holding; a single current snapshot is not enough.
- **Needs additional entity** - requires a new table/relationship beyond
  holdings (an accounts table, or a target-allocation table), not just new
  columns.
- **Not a data gap / needs external data source** - these two widgets are
  out of reach for a data-only effort: one needs a CRUD feature (taxonomy
  management), the other needs a data source this feature's own holdings
  table was never meant to carry (fund constituents).
