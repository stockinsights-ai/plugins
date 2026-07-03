---
name: screen-financial-metrics
description: Screen, fetch, or compare Indian listed companies on income statement, balance sheet, and cash flow metrics from XBRL filings. Use when a query asks to find or rank companies by financial-statement criteria (revenue, net profit, borrowings, cash flow, EPS, solvency ratios), or to fetch/compare those metrics for specific companies by ticker, optionally over a fiscal period or window.
---

# Screen Financial Metrics

## Purpose

Screen and rank Indian listed companies on financial-statement metrics sourced from XBRL filings (income statement, balance sheet, cash flow). Conditions can span all three statements, be combined with AND/OR, be evaluated over a fiscal period or multi-period window, and be restricted to specific companies by ticker. Also used to fetch or compare one or more companies' statement metrics.

## Example Queries

- Companies with consolidated revenue over 5000 crore in every one of the last 5 years.
- Companies with net profit over 1000 crore AND debt-to-equity below 1.
- Companies with FY26 revenue over 200000 crore OR net profit over 30000 crore.
- What is TCS revenue over the last 5 years?
- Compare net profit of TCS and Infosys in FY26.

## Data

Curated financial-statement metrics derived from company XBRL filings. Amounts are in **INR crore**; `eps_*` and `face_value` are in INR per share; `*_ratio` and `return_on_assets` are plain numbers; `gross_npa_pct` is a percent.

- **Income statement**: `revenue`, `total_income`, `other_income`, `total_expenses`, `cost_of_materials`, `employee_cost`, `finance_costs`, `depreciation`, `other_expenses`, `pbt`, `exceptional_items`, `tax_expense`, `net_profit`, `net_profit_owners`, `comprehensive_income`, `eps_basic`, `eps_diluted`, `face_value`, `debt_equity_ratio`, `interest_coverage_ratio`, `debt_service_coverage_ratio`, `return_on_assets`, `gross_npa_pct`
- **Balance sheet**: `total_assets`, `total_equity`, `equity_share_capital`, `reserves_and_surplus`, `borrowings_current`, `borrowings_noncurrent`, `total_borrowings`, `current_assets`, `current_liabilities`, `inventories`, `cash_and_equivalents`, `property_plant_equipment`, `deposits`, `advances`
- **Cash flow**: `cash_from_operations`, `cash_from_investing`, `cash_from_financing`, `net_change_in_cash`

Variations to expect: a value can be `null` when a company did not report that metric for a period; bank/NBFC-specific metrics (`gross_npa_pct`, `return_on_assets`, `deposits`, `advances`) are only populated for financial companies.

Not available here: market/valuation ratios (PE, PS, EV/EBITDA, ROE, ROCE). Market cap is not a screenable metric — resolve a market-cap/sector/industry universe to tickers first (see company-data skill), then pass those tickers.

## Sources and Tools

- Market: `in`
- MCP server: `stockinsights-in`
- Primary tool: `screen_financial_statements` — runs the screen / fetch / comparison.
- Discovery tool: `list_financial_statement_metrics` — returns the authoritative metric catalog (keys, units, source codes); takes no input. Call it only if unsure of valid metric keys.
- Required first step for sector / industry / market-cap universes: resolve them to tickers with the `company-data` skill (`filter_companies`), then pass the tickers here. This tool has no sector/market-cap filter — only `tickers`.

Use only the `stockinsights-in` MCP server. If a tool is unavailable, report the blocker plainly.

## Input Guidance

Call `screen_financial_statements` with:

- `condition_groups`: screening logic in **disjunctive normal form** — conditions **within a group** are AND-ed; **groups** are OR-ed. Each condition is `{ metric, operator, value }` (operator one of `>`, `<`, `>=`, `<=`, `=`; amount values in INR crore).
  - Pure AND (`A AND B`): one group → `[{ "conditions": [A, B] }]`.
  - Pure OR (`A OR B`): one condition per group → `[{ "conditions": [A] }, { "conditions": [B] }]`.
  - Mixed (`(A AND B) OR C`): `[{ "conditions": [A, B] }, { "conditions": [C] }]`.
- `metrics`: extra metric keys to include in the output **without screening** (for fetch/compare).
- `tickers`: plain (`TCS`) or exchange-qualified (`NSE:RELIANCE`, `BSE:500325`); OR within the list. Required when there are no `condition_groups` (a pure lookup must be bounded).
- `period`: `{ reporting_type, mode, ... }` (see below). Defaults to latest annual.
- `match`: how a condition is applied across multiple periods — `latest` (default), `all` (every period), `any` (at least one), `average` (mean).
- `statement_scope`: `consolidated` (default) or `standalone`. `audit_status`: `any` (default), `audited`, `unaudited`.
- `sort` (`{ metric, direction }`) and `limit` (default 50).

At least one of `condition_groups` or `metrics` is required. To rank a universe, use a broad condition plus `sort`.

### Period

`period.reporting_type` sets the **granularity**:

- `quarterly`: standalone-quarter figure. Most companies are mandated to file quarterly, so this suits most recent-period screens.
- `annual`: full-year figure (this is the tool's fallback when `reporting_type` is omitted).
- `half_yearly`: H1 cumulative figure.
- `nine_months`: 9M cumulative figure (cash-flow 9M is essentially not filed — expect nulls).

`period.mode` selects **which periods** of that granularity:

- `latest`: the latest available period of `reporting_type` (for `quarterly`, the actual latest reported quarter — not hardcoded to Q4).
- `all`: every available period.
- `last`: the last N periods; set `last` to a number (e.g. `reporting_type: quarterly` + `last: 4` = last 4 quarters).
- `periods`: explicit periods — `fiscal_years` (`FYxx`, e.g. `FY26`) for `annual`/`half_yearly`/`nine_months`, **or** `fiscal_quarters` (`FYxxQy`, e.g. `FY26Q4`) for `quarterly`.

If a screen returns no data, retry with a different `reporting_type` (e.g. `half_yearly` or `annual`) — some companies do not file quarterly.

### Payload examples

Revenue over 5000 cr in every one of the last 5 years (pure AND, annual):

```json
{
  "condition_groups": [
    { "conditions": [{ "metric": "revenue", "operator": ">", "value": 5000 }] }
  ],
  "period": { "reporting_type": "annual", "mode": "last", "last": 5 },
  "match": "all",
  "sort": { "metric": "revenue", "direction": "desc" }
}
```

FY26 revenue over 200000 cr OR net profit over 30000 cr (pure OR):

```json
{
  "condition_groups": [
    {
      "conditions": [{ "metric": "revenue", "operator": ">", "value": 200000 }]
    },
    {
      "conditions": [
        { "metric": "net_profit", "operator": ">", "value": 30000 }
      ]
    }
  ],
  "period": {
    "reporting_type": "annual",
    "mode": "periods",
    "fiscal_years": ["FY26"]
  }
}
```

Mixed — (revenue > 5000 AND net_profit > 500) OR cash_from_operations > 2000, FY26:

```json
{
  "condition_groups": [
    {
      "conditions": [
        { "metric": "revenue", "operator": ">", "value": 5000 },
        { "metric": "net_profit", "operator": ">", "value": 500 }
      ]
    },
    {
      "conditions": [
        { "metric": "cash_from_operations", "operator": ">", "value": 2000 }
      ]
    }
  ],
  "period": {
    "reporting_type": "annual",
    "mode": "periods",
    "fiscal_years": ["FY26"]
  }
}
```

Single-company lookup — TCS revenue and net profit over the last 5 years (no conditions; metrics + ticker):

```json
{
  "metrics": ["revenue", "net_profit"],
  "tickers": ["TCS"],
  "period": { "reporting_type": "annual", "mode": "last", "last": 5 }
}
```

Compare — net profit of TCS and Infosys in FY26:

```json
{
  "metrics": ["net_profit"],
  "tickers": ["TCS", "INFY"],
  "period": {
    "reporting_type": "annual",
    "mode": "periods",
    "fiscal_years": ["FY26"]
  },
  "sort": { "metric": "net_profit", "direction": "desc" }
}
```

TCS quarterly revenue — last 4 standalone quarters:

```json
{
  "metrics": ["revenue"],
  "tickers": ["TCS"],
  "period": { "reporting_type": "quarterly", "mode": "last", "last": 4 }
}
```

## Retrieval Workflow

1. Parse intent: which metric conditions, output metrics, period/granularity, scope, and company universe.
2. If the query targets a sector / industry / market-cap universe, resolve it to tickers first via the `company-data` skill, then pass those `tickers`.
3. If unsure of valid metric keys or units, call `list_financial_statement_metrics`.
4. Build the `screen_financial_statements` payload from the constraints above. Use `condition_groups` for screening logic (AND/OR/mixed), `metrics` + `tickers` for a lookup/comparison.
5. Call `screen_financial_statements`. If no rows return, retry with a different `reporting_type` (`half_yearly` or `annual`) before concluding no data.
6. When comparing recent quarterly numbers of multiple companies, note that a company might not have released numbers yet. Inform the same to the user.
7. Some companies might report statements `half_yearly` only and not quarterly. So, if a company does not have `quarterly` data, retry with `half_yearly`.

## Response Guidance

The response has `period` (evaluated `reporting_type` and `fiscal_periods` labels), the applied `condition_groups`/`metrics`, `count`, and `results`. Each result has company identity fields (`company_name`, `ticker`, `industry_sector`, `industry`, `market_cap_cr`) and a `metrics` map; per metric, `values_by_period` (keyed by fiscal label like `FY26`, `FY26Q4`, `FY26H1`) and `value_used` (the value the match mode compared and sorted on).

- Report amount metrics in INR crore.
- A `null` metric value means that company did not report it for that period — do not infer a value.
- If `count` is 0, say so plainly and suggest a different `reporting_type` or period if relevant.

## Nuances

- Use only the `stockinsights-in` MCP server; do not substitute another source or model memory for financial values.
- No ticker + no `condition_groups` is invalid — a lookup must be bounded by `tickers`.
- Valuation metrics (PE, PS, EV/EBITDA, ROE, ROCE) and market-cap screening are out of scope here; route those through the `company-data` skill.
- Balance-sheet metrics are point-in-time and are read at the matching period end regardless of `reporting_type`.
