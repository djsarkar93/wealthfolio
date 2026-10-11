# Feature Source Map

This guide maps each feature on the
[Wealthfolio features page](https://wealthfolio.app/features/) to the source
files that implement it. Use it as a starting point when tracing or changing a
feature.

Verified against commit `ace5c5243`. Paths are relative to the repository root.

## Request flow

Most features follow the same path:

1. A page or feature component in `apps/frontend/src/` calls an adapter.
2. The adapter calls a backend command by name:
   - Shared adapters live in `apps/frontend/src/adapters/shared/`.
   - Runtime-specific adapters live in `apps/frontend/src/adapters/tauri/` and
     `apps/frontend/src/adapters/web/`.
   - Some features own their adapters, such as
     `apps/frontend/src/features/spending/adapters/`.
3. The command reaches the backend through one of two runtimes:
   - **Desktop:** a Tauri command in `apps/tauri/src/commands/`, registered in
     `apps/tauri/src/lib.rs`.
   - **Web:** an Axum route in `apps/server/src/api/`. The frontend maps command
     names to routes in `apps/frontend/src/adapters/web/core.ts`.
4. Both runtimes call the same domain logic, mostly in `crates/core/src/`.
5. Domain logic reads and writes SQLite through repositories in
   `crates/storage-sqlite/src/`. The schema is in
   `crates/storage-sqlite/src/schema.rs`, and its history is in
   `crates/storage-sqlite/migrations/`.

Some features have no table of their own. Income, net worth, insights, and
performance are calculated from activities, holdings, quotes, snapshots,
valuations, and FX rates.

## Investment tracking and CSV import

- **Frontend:** `apps/frontend/src/pages/activity/`,
  `apps/frontend/src/pages/activity/import/`,
  `apps/frontend/src/pages/holdings/`, `apps/frontend/src/pages/account/`
- **Adapters:** `apps/frontend/src/adapters/shared/activities.ts`,
  `apps/frontend/src/adapters/shared/portfolio.ts`
- **Core:** `crates/core/src/activities/` (CSV parsing in
  `crates/core/src/activities/csv_parser.rs`), `crates/core/src/accounts/`,
  `crates/core/src/portfolio/holdings/`
- **Desktop:** `apps/tauri/src/commands/activity.rs` (`parse_csv`,
  `check_activities_import`, `import_activities`),
  `apps/tauri/src/commands/account.rs`, `apps/tauri/src/commands/portfolio.rs`
- **Web:** `apps/server/src/api/activities.rs` (`/activities/import`,
  `/activities/import/check`, `/activities/import/parse`),
  `apps/server/src/api/accounts.rs`, `apps/server/src/api/holdings/`
- **Storage:** `crates/storage-sqlite/src/activities/` (also saves import
  profiles and templates), `crates/storage-sqlite/src/accounts/`
- **Migrations:** `2024-10-15-173026_csv_import_profiles`,
  `2025-06-11-133126_account_import_mapping`,
  `2026-01-24-000001_improve_import_profiles`,
  `2026-03-19-000001_import_templates`

## Net worth tracking

- **Frontend:** `apps/frontend/src/pages/net-worth/`
- **Hook:** `apps/frontend/src/hooks/use-alternative-assets.ts`
- **Adapter:** `apps/frontend/src/adapters/shared/alternative-assets.ts`
- **Core:** `crates/core/src/portfolio/net_worth/`,
  `crates/core/src/assets/alternative_assets_service.rs`
- **Desktop:** `apps/tauri/src/commands/alternative_assets.rs`
  (`get_net_worth`, `get_net_worth_history`)
- **Web:** `apps/server/src/api/net_worth.rs` (`/net-worth`,
  `/net-worth/history`), `apps/server/src/api/alternative_assets.rs`
- **Storage:** `crates/storage-sqlite/src/assets/alternative_repository.rs`,
  `crates/storage-sqlite/src/portfolio/valuation/`,
  `crates/storage-sqlite/src/portfolio/snapshot/`

Net worth is calculated from accounts, assets, snapshots, quotes, valuations, FX
rates, and alternative assets.

## Spending and budgets

- **Frontend:** `apps/frontend/src/features/spending/`,
  `apps/frontend/src/pages/settings/spending/`
- **Adapters:** `apps/frontend/src/features/spending/adapters/` (budget,
  reports, insights, events, cash activities, rules, settings).
  `apps/frontend/src/adapters/shared/spending.ts` covers only settings and
  categorization rules.
- **Domain:** `crates/spending/src/`
- **Desktop:** `apps/tauri/src/commands/spending.rs`
- **Web:** `apps/server/src/api/spending.rs`
- **Storage:** `crates/storage-sqlite/src/spending/`
- **Migrations:** `2026-05-25-000001_spending_module`,
  `2026-06-28-000001_spending_activity_splits`,
  `2026-08-09-000001_rule_amount_condition`

## Allocation targets and rebalancing

- **Frontend:** `apps/frontend/src/pages/allocation-targets/`. The
  `/allocation-targets` route redirects to `/insights?tab=overview`.
- **Adapter:** `apps/frontend/src/adapters/shared/allocation-targets.ts`
- **Core:** `crates/core/src/portfolio/allocation_targets/`
- **Desktop:** `apps/tauri/src/commands/allocation_targets.rs`
  (`get_allocation_target_drift`, `calculate_rebalance_plan`)
- **Web:** `apps/server/src/api/allocation_targets.rs`
- **Storage:** `crates/storage-sqlite/src/portfolio/allocation_targets/`
- **Migrations:** `2026-05-25-000002_allocation_targets`,
  `2026-06-22-000001_hybrid_drift_bands`,
  `2026-06-25-000001_allocation_constraints`

## AI assistant

- **Frontend:** `apps/frontend/src/pages/ai-assistant/`,
  `apps/frontend/src/features/ai-assistant/` (streaming client in
  `apps/frontend/src/features/ai-assistant/api/stream.ts`)
- **Adapters:** `apps/frontend/src/adapters/shared/ai-threads.ts`,
  `apps/frontend/src/adapters/shared/ai-providers.ts`
- **AI logic:** `crates/ai/src/`, with tools in `crates/agent-tools/src/`
- **Desktop:** `apps/tauri/src/commands/ai_chat.rs` (`stream_ai_chat`, which
  streams over a Tauri channel), `apps/tauri/src/commands/ai_providers.rs`
- **Web:** `apps/server/src/api/ai_chat.rs` (`POST /ai/chat/stream`),
  `apps/server/src/api/ai_providers.rs`
- **Storage:** `crates/storage-sqlite/src/ai_chat/`
- **Migration:** `2026-01-15-000001_ai_chat_persistence`

## Portfolio insights

- **Frontend:** `apps/frontend/src/pages/insights/`,
  `apps/frontend/src/pages/holdings/`
- **Hook:** `apps/frontend/src/hooks/use-portfolio-allocations.ts`
- **Adapter:** `apps/frontend/src/adapters/shared/portfolio.ts`
  (`getPortfolioAllocations`, `getHoldingsByAllocation`)
- **Core:** `crates/core/src/portfolio/allocation/allocation_service.rs`
  (built from holdings and taxonomies), `crates/core/src/taxonomies/`
- **Desktop:** `apps/tauri/src/commands/portfolio.rs`
  (`get_portfolio_allocations`, `get_holdings_by_allocation`),
  `apps/tauri/src/commands/taxonomy.rs`
- **Web:** `apps/server/src/api/holdings/` (`/allocations/query`),
  `apps/server/src/api/taxonomies.rs`
- **Storage:** `crates/storage-sqlite/src/taxonomies/`,
  `crates/storage-sqlite/src/assets/`
- **Migration:** `2026-01-01-000002_taxonomies`

## Performance dashboard

- **Frontend:** `apps/frontend/src/pages/performance/`; custom portfolios in
  `apps/frontend/src/pages/settings/portfolios/`
- **Adapters:** `apps/frontend/src/adapters/shared/portfolio.ts`,
  `apps/frontend/src/adapters/shared/portfolios.ts`
- **Core:** `crates/core/src/portfolio/performance/`,
  `crates/core/src/portfolios/`
- **Calculation engine:** `crates/portfolio-engine/src/`, used through the core
  performance, valuation, snapshot, and coordinator services
- **Desktop:** `apps/tauri/src/commands/portfolio.rs`
  (`calculate_performance_history`, `calculate_performance_summary`),
  `apps/tauri/src/commands/portfolios.rs`
- **Web:** `apps/server/src/api/performance.rs` (`/performance/history`,
  `/performance/summary`, `/performance/summaries`),
  `apps/server/src/api/portfolios.rs`
- **Storage:** `crates/storage-sqlite/src/portfolio/valuation/`,
  `crates/storage-sqlite/src/portfolio/snapshot/`,
  `crates/storage-sqlite/src/portfolios/`,
  `crates/storage-sqlite/src/market_data/`
- **Migration:** `2026-05-11-000001_portfolios`

## Income tracking

- **Frontend:** `apps/frontend/src/pages/income/`
- **Adapter:** `apps/frontend/src/adapters/shared/portfolio.ts`
  (`getIncomeSummary`)
- **Core:** `crates/core/src/portfolio/income/income_service.rs`
- **Desktop:** `apps/tauri/src/commands/portfolio.rs` (`get_income_summary`)
- **Web:** `apps/server/src/api/performance.rs` (`/income/summary`,
  `/income/summary/query`)
- **Storage:** `crates/storage-sqlite/src/activities/`

Income is calculated from recorded activities and FX rates.

## Goals and save-up planner

- **Frontend:** `apps/frontend/src/features/goals/`,
  `apps/frontend/src/features/goals/save-up/`
- **Adapter:** `apps/frontend/src/adapters/shared/goals.ts`
- **Core:** `crates/core/src/goals/`, `crates/core/src/planning/save_up.rs`
- **Desktop:** `apps/tauri/src/commands/goal.rs` (including
  `get_save_up_overview`)
- **Web:** `apps/server/src/api/goals.rs`
- **Storage:** `crates/storage-sqlite/src/goals/`
- **Migration:** `2026-03-30-000001_goals_and_retirement_planning`

## Retirement and FIRE planner

- **Frontend:** `apps/frontend/src/features/goals/retirement-planner/`
- **Adapters:**
  - The overview uses `apps/frontend/src/adapters/shared/goals.ts`
    (`getRetirementOverview`).
  - Projections and simulations use runtime-specific adapters:
    `apps/frontend/src/adapters/tauri/fire-planner.ts` and
    `apps/frontend/src/adapters/web/fire-planner.ts`.
- **Core:** `crates/core/src/planning/retirement/`,
  `crates/core/src/portfolio/fire/`
- **Desktop:**
  - `apps/tauri/src/commands/goal.rs` has the overview.
  - `apps/tauri/src/commands/fire.rs` has projection, Monte Carlo, stress tests,
    scenario analysis, sequence-of-returns risk, and the sensitivity map.
- **Web:** `apps/server/src/api/goals.rs` (`/goals/retirement/*`)
- **Storage:** `crates/storage-sqlite/src/goals/` (goal plans)
- **Migration:** `2026-03-30-000001_goals_and_retirement_planning`

## Contribution room and limit tracking

- **Frontend:** `apps/frontend/src/pages/settings/contribution-limits/`
- **Adapter:** `apps/frontend/src/adapters/shared/contribution-limits.ts`
- **Core:** `crates/core/src/limits/`
- **Desktop:** `apps/tauri/src/commands/limits.rs`
- **Web:** `apps/server/src/api/limits.rs` (`/limits`, `/limits/{id}`,
  `/limits/{id}/deposits`)
- **Storage:** `crates/storage-sqlite/src/limits/`
- **Migrations:** `2024-10-08-193300_contrib_limits`,
  `2025-03-17-185736_add_start_end_dates_to_contribution_limits`

## Changing a feature

- When you add or change a backend call, update both runtimes: the frontend
  adapter, the Tauri command and its registration in `apps/tauri/src/lib.rs`,
  the web mapping in `apps/frontend/src/adapters/web/core.ts`, and the Axum
  route. Check the wiring with
  `apps/frontend/src/adapters/adapter-command-parity.test.ts`.
- Keep Tauri commands and Axum handlers thin. Put business logic in the owning
  crate.
- Add schema changes as a new migration in `crates/storage-sqlite/migrations/`.
- Never edit a migration that has already shipped.
