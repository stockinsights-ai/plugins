# Screen Financial Metrics

## Purpose

Query, compare, screen, and rank Indian listed companies across XBRL statements - income, balance-sheet, cash-flow, segment-revenues - and current company data. SQL may join all five tables, so conditions and derived metrics may span them in one query.

Use `filings-search` for absent company-defined KPIs such as order book, capacity, utilization, ARPU, volumes, or presentation segment labels.

## Source and Tool

- Market: `in`
- SQL dialect: PostgreSQL
- Tool: `query_structured_financial_data`
- Default result limit: 50 rows unless the analyst requests otherwise.

This reference selects `query_structured_financial_data` as the primary source; do not substitute another source before querying it. Use these payload shapes in order:

1. Inspect every relevant table in one call: `{ "body": { "action": "describe_tables", "table_names": ["in_companies", "in_xbrl_income_statements"] } }`. Never inspect the same table twice.
2. Execute: `{ "body": { "action": "run_query", "query": "SELECT ... LIMIT 50", "max_rows": 50 } }`. Use `query`, not `sql`; omitted `max_rows` defaults to 100.

Double-check SQL. Correct one input-shape rejection from these examples; after a second, stop and report an internal tool-input failure. Retry an API SQL error only when its details identify the query problem. Never search filings for tool documentation.

Use read-only SQL, escaped literals, explicit columns, and no `$1` placeholders. Answer only from returned data.

## Primary-Source Fallback

If SQL is available but the requested reported fact is absent, read `references/datasets/filing-content.md` and retrieve the matching quarterly result. If that filing is unavailable or incomplete, read `references/datasets/filings-search.md` and use the matching investor presentation. Preserve the requested period, say which primary source was missing, and identify the fallback as company-reported filing or presentation data rather than a standardised SQL result.

If SQL fails because of an input or query error, follow its correction rules before falling back. Never use a fallback to conceal a query that was not attempted or could still be corrected.

## Tables

All tables are in `public`.

| Table                          | Use                                              |
| ------------------------------ | ------------------------------------------------ |
| `in_companies`                 | Identity, industry, and current market snapshot. |
| `in_xbrl_income_statements`    | Income-statement flows and disclosed ratios.     |
| `in_xbrl_balance_sheets`       | Point-in-time balance-sheet metrics.             |
| `in_xbrl_cash_flow_statements` | Cash-flow metrics.                               |
| `in_xbrl_segment_revenues`     | Business/geographical segment financial facts.   |

Join XBRL tables on `in_companies.company_id`. Its ticker column is `stock_ticker`, never `ticker`. Reduce each fact table to one row per company-period before joining; raw fact-to-fact joins multiply rows.

### Common XBRL columns

The XBRL tables share `xbrl_filing_id`, `company_id`, period fields, `reporting_type`, `statement_scope`, `audit_status`, `metric_group`, `metric_code`, `component_key`, `is_total`, `value`, and `unit`.

Select practical latest filings with `MAX(xbrl_filing_id)` by company and period, then join the id back before pivoting. Never use `MAX(value)` across filings.

For company-level totals, normally require:

```sql
is_total = true
AND NULLIF(TRIM(component_key), '') IS NULL
```

Follow `company-data.md` for `in_companies` and peer universes; its market cap, prices, and PE are current snapshots.

## Metric Catalog

XBRL amounts are stored in INR. Divide amount facts by `10000000` for INR crore. Do not scale EPS, face value, ratios, or percentages; check `unit` and `metric_group` when uncertain. Preserve fallback-code priority with `COALESCE(MAX(value) FILTER (...), ...)`; never sum alternatives.

### Income statement codes

| Metric | `metric_code` priority |
|---|---|
| Revenue | `revenue_from_operations`, then `income` |
| Total income | `income` |
| Other income | `other_income` |
| Total expenses | `expenses` |
| Cost of materials | `cost_of_materials_consumed` |
| Employee cost | `employee_benefit_expense`, then `employees_cost` |
| Finance costs | `finance_costs`, then `interest_expended` |
| Depreciation and amortisation | `depreciation_depletion_and_amortisation_expense`, then `depreciation_and_amortisation_expense` |
| Other expenses | `other_expenses` |
| Profit before tax | `profit_before_tax`, `profit_loss_from_ordinary_activities_before_tax`, then `profit_before_extraordinary_items_and_tax` |
| Exceptional items | `exceptional_items_before_tax`, then `exceptional_items` |
| Tax expense | `tax_expense` |
| Net profit | `profit_loss_for_period`, `profit_loss_for_the_period`, `profit_loss_for_period_from_continuing_operations`, then `profit_loss_for_the_period_from_continuing_operations` |
| Net profit attributable to owners | `profit_or_loss_attributable_to_owners_of_parent` |
| Comprehensive income | `comprehensive_income_for_the_period` |
| Basic EPS | `basic_earnings_loss_per_share_from_continuing_and_discontinued_operations`, `basic_earnings_loss_per_share_from_continuing_operations`, then `basic_earnings_per_share_after_extraordinary_items` |
| Diluted EPS | `diluted_earnings_loss_per_share_from_continuing_and_discontinued_operations`, `diluted_earnings_loss_per_share_from_continuing_operations`, then `diluted_earnings_per_share_after_extraordinary_items` |
| Face value | `face_value_of_equity_share_capital` |
| Disclosed debt/equity | `debt_equity_ratio` |
| Disclosed interest coverage | `interest_service_coverage_ratio` |
| Disclosed debt-service coverage | `debt_service_coverage_ratio` |
| Bank/NBFC return on assets | `return_on_assets` |
| Bank/NBFC gross NPA % | `percentage_of_gross_npa` |

### Balance-sheet codes

| Metric | `metric_code` priority |
|---|---|
| Total assets | `assets` |
| Total equity | `equity` |
| Equity share capital | `equity_share_capital`, then `capital` |
| Reserves and surplus | `reserves_and_surplus`, then `other_equity` |
| Current borrowings | `borrowings_current` |
| Non-current borrowings | `borrowings_noncurrent` |
| Total borrowings | Prefer `borrowings`; otherwise sum available `borrowings_current` and `borrowings_noncurrent` |
| Current assets | `current_assets` |
| Current liabilities | `current_liabilities` |
| Inventories | `inventories` |
| Cash and equivalents | `cash_and_cash_equivalents` |
| Property, plant and equipment | `property_plant_and_equipment` |
| Bank deposits | `deposits` |
| Bank advances | `advances` |

### Cash-flow codes

| Metric | `metric_code` |
|---|---|
| Cash from operations | `cash_flows_from_used_in_operating_activities` |
| Cash from investing | `cash_flows_from_used_in_investing_activities` |
| Cash from financing | `cash_flows_from_used_in_financing_activities` |
| Net change in cash | `increase_decrease_in_cash_and_cash_equivalents` |

### Segment codes

`in_xbrl_segment_revenues` stores business or geographical facts, not company totals.

| Measure | Common `metric_code` |
|---|---|
| Revenue | `segment_revenue`, `segment_revenue_from_operations` |
| Inter-segment revenue | `inter_segment_revenue` |
| Profit | `segment_profit_loss_before_tax_and_finance_costs`, `segment_profit_before_tax` |
| Finance costs | `segment_finance_costs` |
| Assets | `segment_assets`, `net_segment_assets` |
| Liabilities | `segment_liabilities`, `net_segment_liabilities` |
| Unallocated balances | `un_allocable_assets`, `un_allocable_liabilities` |

For operating components normally require `is_total = false` and a non-empty `component_key`; totals and reconciliations may use `is_total = true` and an empty key. Group by company, period, metric, and component. Raw component keys can change across companies or years and can represent eliminations or unallocated items; do not equate them without evidence. For revenue mix use positive operating rows and exclude eliminations. Revenue, profit, and finance costs are duration facts; assets and liabilities are point-in-time.

A requested metric absent from this catalogue may be derived rather than missing, so follow Metric Definition Routing before probing the table. Only when neither this catalogue nor the relevant definition reference classifies it, run one narrow, bounded distinct-values query returning only `metric_code`, `metric_group`, and `unit`; do not guess.

## Period and Filing Rules

- Annual income/cash flow: `reporting_type = 'annual'`, normally `fiscal_quarter = 'Q4'`.
- Standalone quarter flow: `reporting_type = 'quarterly'` with the requested fiscal quarter.
- Half-year cumulative flow: `reporting_type = 'half_yearly'`, normally Q2/H1.
- Nine-month cumulative flow: `reporting_type = 'nine_months'`; cash-flow coverage is sparse.
- Balance sheet: `reporting_type = 'as_of'`; it is a point-in-time value. Q2 and Q4 are generally better covered than Q1 and Q3.
- Default statement scope: `LOWER(TRIM(statement_scope)) = 'consolidated'`. Use standalone only when requested or consolidated data is explicitly unavailable and the analyst accepts the substitution.
- Apply `audit_status` only when requested. If not filtered, do not characterize the returned rows as audited or unaudited.

Always constrain both `reporting_type` and fiscal period. Q4 alone is ambiguous: it can identify a standalone fourth quarter or a full annual result.

Use an explicit period exactly; report missing data rather than substituting. For “latest,” determine available periods first and prefer a common comparison period. Otherwise return each period and disclose the mismatch.

Use identical period, statement scope, audit status, and metric definitions across compared companies. Do not mix quarterly flows with annual or point-in-time balances.

## Metric Definition Routing

Before constructing SQL, read only the definition references required by the requested metrics:

- Operating profit, OPM, operating EBITDA, EBIT, margins, tax, expenses, and per-share calculations: `references/datasets/financial-metrics/profitability-and-margins.md`
- Growth, CAGR, TTM, averages, medians, period trends, and margin changes: `references/datasets/financial-metrics/growth-and-trends.md`
- ROE, ROA, ROCE, ROIC, turnover, and financial leverage: `references/datasets/financial-metrics/returns-and-efficiency.md`
- Liquidity, debt, coverage, working capital, and capital structure: `references/datasets/financial-metrics/liquidity-and-leverage.md`
- Cash-flow ratios, earnings quality, accruals, and cash conversion: `references/datasets/financial-metrics/cash-flow-and-quality.md`
- Bank and NBFC-specific metrics: `references/datasets/financial-metrics/banking-metrics.md`
- Unclear, unsupported, or provider-specific metric names: `references/datasets/financial-metrics/metric-coverage-and-routing.md`

A request can require more than one definition reference. For example, “quarterly OPM trend” requires both `profitability-and-margins.md` for the OPM definition and `growth-and-trends.md` for period alignment and trend handling.

Do not construct SQL until the required definition references have been read. Do not guess metric codes or formulas. OPM is derived; do not search for or guess a disclosed OPM code.

## SQL Construction

Use staged CTEs: select each table's latest requested filing, join it to facts, and pivot requested codes into one row per company and period. Join reduced CTEs and `in_companies`, then calculate screens and ratios in an outer CTE. Select answer columns with deterministic ordering and `LIMIT`.

Use `NULLIF(denominator, 0)` for division. Preserve missing facts as `NULL`. For borrowings, prefer a reported total; sum current and non-current only as fallback.

Calculated metrics may be filtered, sorted, and ranked in an outer SQL query when every company uses the same definition and inputs. Prefer a disclosed ratio only when the statement metric catalogue explicitly maps its code; otherwise use the relevant guarded derivation. Label calculated alternatives as `Derived` and state the adopted definition when definitions can differ.

## Retrieval Workflow

1. Parse the universe, metrics, conditions, period, scope, and ranking; resolve only ambiguous names.
2. Read each required metric definition reference.
3. Inspect the database and each required table once. For “latest,” also determine available periods.
4. Write, check, and execute a bounded query under the rules above (`max_rows` 50 by default or the requested count).
5. Repair actionable SQL errors without test queries. For empty latest data, inspect availability; never replace an explicit period.
6. Answer from returned rows, preserving period, scope, units, and missing values.

## Response Guidance

- Report amount metrics in INR crore after SQL scaling.
- Report EPS and face value in INR per share, and ratios/percentages in their natural units.
- `NULL` means the company did not report that fact for the selected filing and period; do not infer zero.
- State the fiscal period, reporting granularity, and statement scope.
- For current company fields such as price, market cap, and PE, state that they are current snapshots when the distinction matters.
- Disclose differing latest periods. Distinguish no match from unavailable data or a failed query.

## Nuances

- Balance-sheet values are point-in-time; never sum them across quarters. Flow granularities are not interchangeable.
- Cash-flow coverage is strongest for annual and half-year periods.
- Bank/NBFC metrics and balance-sheet economics differ from industrial companies; use the banking reference and avoid industrial liquidity formulas for banks.
- `MAX(xbrl_filing_id)` is a latest-filing approximation because the SQL catalog does not expose integrated filing metadata.
- Current price, market cap, PE, and 52-week range are available in `in_companies`; historical market prices and valuation history are not.
