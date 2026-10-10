# Company Data

## Purpose

Resolve Indian listed-company identities and query structured company profiles, current market snapshots, industry universes, and peers.

## Sources and Tools

- Market: `in`
- Company resolution: `resolve_companies` on the `stockinsights-india` MCP server
- Company queries: `run_structured_financial_query` against `public.in_companies`.
- SQL dialect: PostgreSQL
- Default result limit: 50 rows, unless the analyst requests a smaller or specific count

Use `resolve_companies` only for identity resolution. Use `run_structured_financial_query` for every lookup, filter, sort, screen, or peer query against company data.

Before SQL, inspect the relevant tables once with
`describe_structured_financial_tables`, for example
`{"body":{"table_names":["in_companies"]}}`. Then call
`run_structured_financial_query` with a read-only PostgreSQL SELECT in
`body.query` and an explicit `body.max_rows`, for example
`{"body":{"query":"SELECT company_id, company_name, stock_ticker FROM public.in_companies WHERE stock_ticker = 'TCS' ORDER BY company_id LIMIT 5","max_rows":5}}`.
Use `query`, not `sql`; the tools have separate input schemas. Select only
confirmed columns, use qualified `public` table names, escape literals, and
include deterministic ordering and LIMIT. Never use `SELECT *` or unbound
placeholders. Correct an input rejection once from the tool schema; use SQL
error details and inspected metadata to repair a query. A transport error is
not an empty result, and an empty result under unverified filters is not proof
that the company is absent.

## Company Resolution

Use `resolve_companies` to resolve one or more company names or partial identifiers into confirmed ticker symbols and company details. It returns up to five candidate matches per entity, ranked by confidence.

Use it when:

- A name rather than an obvious ticker is given, such as “ICICI Bank,” “Zomato,” or “Reliance.”
- A company name or identifier is ambiguous.
- A confirmed ticker or `company_id` is needed before querying another dataset.

Do not resolve obvious tickers or use resolution to discover a broad sector, industry, or market-cap universe.

## `in_companies`

Confirm the live schema before querying. Commonly relevant columns are:

| Column                           | Meaning                                                                  |
| -------------------------------- | ------------------------------------------------------------------------ |
| `company_id`                     | Stable internal company identifier and the join key for other datasets.  |
| `company_name`                   | Company display name.                                                    |
| `company_website`                | Company website URL.                                                     |
| `stock_ticker`                   | Primary plain Indian stock ticker, such as `TCS` or `RELIANCE`.          |
| `isin`                           | International Securities Identification Number.                          |
| `bse_id`, `bse_ticker`, `nse_id` | Exchange-specific identifiers and tickers.                               |
| `marketcap`                      | Current market capitalization in **INR crore**.                          |
| `marketcap_category`             | Stored size bucket: exactly `large`, `mid`, `small`, `micro`, or `nano`. |
| `current_price`                  | Current share price in INR.                                              |
| `high_52w`, `low_52w`            | Current 52-week high and low prices in INR.                              |
| `pe_ratio`                       | Current price-to-earnings ratio; a plain ratio.                          |
| `industry_macro`                 | Broadest industry classification.                                        |
| `industry_sector`                | Sector within the macro classification.                                  |
| `industry`                       | Industry within the sector.                                              |
| `industry_basic`                 | Most specific industry classification.                                   |
| `company_info`                   | JSON metadata such as currency and location.                             |
| `company_links`                  | JSON collection of company-related links.                                |
| `previous_tickers`               | JSON history of former ticker symbols.                                   |

Market cap, price fields, PE, and 52-week values are current snapshots, not historical values; do not attach a historical fiscal period to them.

The table does not provide historical prices or valuations, PB, PS, PEG, EV/EBITDA, technical indicators, forecasts, ownership, or shareholding data. Do not manufacture unsupported fields.

## Query Guidance

- Exact company: filter by the confirmed uppercase `stock_ticker`.
- Named list: use `IN (...)` with confirmed tickers.
- Market-cap category: use only `large`, `mid`, `small`, `micro`, or `nano`; do not infer a category from `marketcap`.
- Numeric screens: compare `marketcap`, `current_price`, `high_52w`, `low_52w`, or `pe_ratio` directly and exclude `NULL` where required.
- Default ranking: `ORDER BY marketcap DESC NULLS LAST`.
- Default bound: `LIMIT 50` and `max_rows: 50`.
- Market-cap proximity: order by `ABS(candidate.marketcap - source.marketcap)`, then market cap descending as a tie-breaker.

For industry terms such as auto, banking, IT, or pharma, read `references/datasets/industry-classification.json`:

- Copy exact field names and values from matching classification rows.
- Do not normalize capitalization, punctuation, or legacy variants.
- Fields within one selected row use AND; multiple selected rows use OR.
- Ask for clarification when matches represent materially different businesses.

### Taxonomy membership versus thematic exposure

An exact sector or industry-membership question is a taxonomy screen. Use only exact rows from the classification reference. If the requested category is absent, the taxonomy cannot answer it.

An exposure, theme, product, geography, customer, end-market, capability, or value-chain question is a business-evidence screen, even when its words resemble an industry label. For these questions:

- Do not approximate a missing category with adjacent classifications, relax to broader industries, or search company names for fragments. A broad label or name match does not prove exposure.
- Read `references/datasets/filings-search.md` and discover companies from their disclosures before applying structured filters.
- After validating candidate tickers, query all of them once in `in_companies` for current P/E, price, market cap, classification, or other available company fields. Apply the analyst's numeric conditions in that query and return only the intersection.
- P/E, price, market cap, and 52-week fields are already in `in_companies`; do not read `screen-financial-metrics.md` unless the question also needs a reported statement metric or calculation.
- Describe filing-discovered results as an evidence-backed shortlist, not an exhaustive market universe.

Select only columns needed for the answer. For example, a sector universe normally needs company name, ticker, the requested classification fields, market cap, and any requested market metric—not website, links, and internal metadata.

## Finding Peers

1. Resolve the source company only if its identity is ambiguous.
2. Query its `company_id`, ticker, all four industry levels, and market cap from `in_companies`.
3. In the same SQL query or a follow-up bounded query, select candidates matching all populated classification levels.
4. Exclude the source company and order candidates by absolute market-cap distance.
5. If too few peers match, relax `industry_basic`, then `industry`, then `industry_sector`. Keep the broadest meaningful classification and state the rule used.
6. Unless requested otherwise, return at most ten peers.

Use a source-company CTE so peer matching and market-cap proximity are evaluated in SQL without copying a possibly `NULL` value into a guessed query.

## Cross-Dataset Use

`in_companies.company_id` joins to the XBRL tables documented in `screen-financial-metrics.md`. Use that reference when a screen or comparison combines company attributes with revenue, profit, balance-sheet, cash-flow, derived, or segment metrics. Do the universe filter and financial joins in one SQL query when practical rather than retrieving a large ticker list and sending it through another tool.

## Output Rules

- Keep the final answer concise and state the selection or peer rule.
- Identify current market and valuation fields as snapshots when relevant.
- Preserve `NULL` as unavailable; never treat it as zero.
- If no rows match, say so plainly.
- If the SQL tool is unavailable or fails after correction, report the blocker rather than using model memory.
