---
name: screen-financial-metrics
description: First-choice skill for Indian financial-statement metric questions. Fetch, compare, screen, or rank companies using XBRL income-statement, balance-sheet, cash-flow, EPS, supported reported ratios, and statement-only derived metrics for explicit or relative fiscal periods. Do not use for market-price/valuation snapshots or company-defined operational KPIs.
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

Market-price and valuation fields such as price, market cap, PE, PS, PB, PEG, and EV/EBITDA are not available here. Resolve a market-cap/sector/industry universe to tickers with `company-data`, then pass those tickers here. For order book, capacity, utilization, ARPU, volumes, segment mix, or other company-defined KPIs, use `filings-search`.

## Derived-Metric Routing

The MCP tool accepts only built-in metric keys. For a requested derived metric, fetch its built-in dependencies and calculate it after retrieval. Read only the relevant reference:

- Profit, EBITDA/EBIT, OPM/NPM, tax, expense, or per-share derivations: `references/profitability-and-margins.md`
- YoY/QoQ growth, CAGR, TTM, averages, medians, or margin change: `references/growth-and-trends.md`
- ROE, ROA, ROCE/ROIC, turnover, or financial leverage: `references/returns-and-efficiency.md`
- Working capital, liquidity, debt, coverage, or capital structure: `references/liquidity-and-leverage.md`
- Cash-flow ratios, earnings quality, or accruals: `references/cash-flow-and-quality.md`
- Bank/NBFC-specific metrics: `references/banking-metrics.md`
- Unclear Screener-style labels or unsupported inputs: `references/metric-coverage-and-routing.md`

Apply these safeguards:

1. Fetch all dependencies in one `screen_financial_statements` call with identical period, statement scope, and audit status.
2. Calculate only when every required value is present and each denominator is non-zero. Never convert a missing value to zero.
3. Use annual flows with average opening/closing balance-sheet values for return and turnover ratios. Do not mix quarterly flows with annual balances.
4. Keep calculations ticker-scoped or limited to returned companies. Never claim an exhaustive screen, filter, or ranking by a derived metric because derived names cannot be sent as conditions or sort keys.
5. Prefer a built-in disclosed ratio when one exists. Label calculated alternatives as `Derived` and state the adopted definition when definitions can differ.

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
- `metrics`: up to 10 built-in metric keys to include in the output **without screening** (for fetch/compare). Never send a derived metric name.
- `tickers`: plain (`TCS`) or exchange-qualified (`NSE:RELIANCE`, `BSE:500325`); OR within the list. Required when there are no `condition_groups` (a pure lookup must be bounded).
- `period`: `{ reporting_type, mode, ... }` (see below). Defaults to latest quarterly.
- `match`: how a condition is applied across the selected periods — `latest` (default; most recent selected period), `all` (every selected period), `any` (at least one selected period), `average` (mean).
- `statement_scope`: `consolidated` (default) or `standalone`. `audit_status`: `any` (default), `audited`, `unaudited`.
- `sort` (`{ metric, direction }`) and `limit` (default 50).

At least one of `condition_groups` or `metrics` is required. To rank a universe, use a broad condition plus `sort`.

### Period

`period.reporting_type` sets the **granularity**:

- `quarterly`: standalone-quarter figure (default; most companies are mandated to file quarterly, so this suits most recent-period screens).
- `annual`: full-year figure.
- `half_yearly`: H1 cumulative figure.
- `nine_months`: 9M cumulative figure (cash-flow 9M is essentially not filed — expect nulls).

`period.mode` selects **which periods** of that granularity:

- `latest`: the latest available period of `reporting_type` (for `quarterly`, the actual latest reported quarter — not hardcoded to Q4).
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
- Market-price and valuation metrics (price, PE, PB, PS, PEG, EV/EBITDA) and market-cap screening are out of scope; route supported fields through `company-data`.
- ROE, ROCE, growth, margins, liquidity ratios, and other supported statement-only calculations are derived after retrieval under the reference rules. They cannot be used directly as MCP conditions, output metric keys, or sort keys.
- Balance-sheet metrics are point-in-time and are read at the matching period end regardless of `reporting_type`.
