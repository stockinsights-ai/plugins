---
name: stock-research
description:
  Research NSE and BSE listed public Indian stocks with authentic financial and filing datasets from stockinsights.ai. Use when
  a user wants to understand a public company's performance, valuation, ownership,
  strategy, risks, or recent developments; compare peers; discover or screen
  companies by financial or business criteria; or investigate management
  commentary, company disclosures, and results dates.
  Use for queries on financial statements and ratios, current market and valuation data,
  company screening and peers, shareholding, and company-reported operational
  metrics. Also for questions that need to be answered by searching filings such as
  annual reports, quarterly results, investor presentations, earnings calls and announcements.

metadata:
  version: "0.1.0"
---

# Equity Research

Answer questions about Indian listed companies with grounded evidence from the `stockinsights-india` MCP server.

## Route Before Retrieving

Choose one primary source before the first MCP call. Read its reference before using that source, and follow its conditional definition and fallback references. Do not call several tools to discover which one works.

| Question is about                                                                                                                        | Read                                              | Primary tool                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Statement metrics: revenue, profit, expenses, debt, assets, cash flow, EPS, statement-derived ratios, segment revenue                    | `references/datasets/screen-financial-metrics.md` | `describe_structured_financial_tables` and `run_structured_financial_query`                                                    |
| Market and valuation fields: market cap, price, 52-week range, PE, sector, industry, peers, simple screens                               | `references/datasets/company-data.md`             | `describe_structured_financial_tables` and `run_structured_financial_query`                   |
| Shareholding: promoter/FII/DII/public mix, named holders, ownership changes                                                              | `references/datasets/shareholding-pattern.md`     | `describe_structured_financial_tables` and `run_structured_financial_query`                                                    |
| Recent filings, filing availability, publication dates                                                                                   | `references/datasets/filings-list.md`             | `list_filings`                                                                       |
| Operational or company-defined KPIs: order book, capacity, utilisation, ARPU, volumes, management-defined segment/product/geographic mix | `references/datasets/filings-search.md`           | `search_filings` with `search_method: "keyword"` on the latest investor presentation |
| Guidance, outlook, management commentary, analyst Q&A                                                                                    | `references/datasets/filings-search.md`           | `search_filings` with `search_method: "semantic"` on the latest earnings transcript  |
| Strategy, business model, long-term risks, cross-company themes                                                                          | `references/datasets/filings-search.md`           | `search_filings` with `search_method: "semantic"` on the latest annual report        |
| Exact phrases, named projects/products, proper nouns, source-native metric labels                                                        | `references/datasets/filings-search.md`           | `search_filings` with `search_method: "keyword"`                                     |
| Recent events: contracts, acquisitions, expansions, regulatory actions, management changes, dividends, rating changes                    | `references/datasets/announcement-feed.md`        | `get_announcements`                                                                  |
| Full summary, detailed analysis, or section extraction from exactly one identified filing                                                | `references/datasets/filing-content.md`           | `get_filing_content`                                                                 |
| Upcoming or latest results dates                                                                                                         | `references/datasets/results-calendar.md`         | `get_results_calendar`                                                               |

Routing rules:

- Before structured SQL, read the routed reference, then inspect the relevant tables. Never guess column names, metric codes, or stored filter values.
- For breakdown, mix, or composition questions, treat the breakdown as the primary intent even when the base metric is a statement metric. Use the latest source that reports the requested dimensions; do not assume category names or document type.
- A question may need two routes, for example a reported metric plus management commentary. Retrieve each part from its own route and combine them in one answer.
- Call `resolve_companies` only when identity is genuinely ambiguous or a confirmed ticker is needed. Do not resolve obvious tickers or use resolution to discover a sector universe.
- For exact industry names read `references/datasets/industry-classification.json`; for announcement categories read `references/datasets/announcement-categories.js`. For metric definitions and calculations, read only the files under `references/datasets/financial-metrics/` that `screen-financial-metrics.md` points to.
- For a follow-up that the conversation's retrieved evidence already answers, answer without a new call.

## Filing Search Fallback

For a company-focused filing question:

1. Search the primary filing with the search mode from the routing table, using ticker and period filters.
2. If evidence is missing or incomplete, try the other search mode when that filing type supports it, using likely source-native labels and synonyms.
3. Treat evidence as insufficient when it is missing, incomplete, or stale relative to a newer reporting period visible in the results.
4. If searches are insufficient, retrieve the most likely filing with `get_filing_content`, then fall back to the alternate filing that `filings-search.md` names. Stop once one source provides complete evidence.

Investor presentations are searchable only by keyword. Quarterly results are not searchable and must be retrieved directly with `get_filing_content`.

## Tool Calls

- Build every payload from the live MCP tool schema. Do not invent fields.
- JSON examples in the references show the contents of the tool's `body` argument; wrap them as `{ "body": { ... } }`. `get_results_calendar` takes `{ "queryParams": { "ticker": "..." } }`.
- Use plain tickers as returned by `resolve_companies`, such as `TCS` or `RELIANCE`. Never add `$` or `NSE:`/`BSE:` prefixes; every tool rejects them. When a name could map to more than one company, resolve it first.
- Use the minimum number of calls. Make additional calls only when a reference documents a fallback or the returned evidence is incomplete.
- If the stockinsights-india tools are unavailable, tell the user to connect or re-authorise the stockinsights.ai India connector, and stop. Do not answer from memory.

## Periods

- If the user specifies a period, use exactly that period. Indian fiscal years run April–March; `FY26Q1` is April–June 2025.
- If the user asks for the latest data or gives no period, select the latest available reported period using the chosen tool.
- Never convert today's date into an explicit filing period. The calendar does not prove a filing exists; let the data source choose the latest reported period.
- If a requested period is unavailable, say so. Do not substitute another period, a consolidated value, or a proxy unless the user asks for it.

## Evidence and Citations

- Use only the `stockinsights-india` MCP server for financial figures. Never use model memory or web search for financial claims.
- Cite filing-backed claims inline with the returned `citation_link`, labelled with company, document type, and fiscal period or publication date. For SQL rows without links, state the source period, units, and stockinsights.ai database provenance.
- Put multiple citations side by side as adjacent links, not in a bundle.
- Never expose internal IDs or citation tokens.
- If evidence is weak, conflicting, incomplete, or unavailable, say what is missing.

## Scope

- Cover only Indian listed companies. For companies listed elsewhere, say the connector does not cover them.
- Do not give buy, sell, or hold recommendations or personalised investment advice. Present the evidence and leave the judgement to the user.

## Response

- Lead with a direct, concise answer in Markdown.
- Report amounts in INR crore unless the user asks otherwise, and state units and periods.
- When the evidence shows a trend, comparison, composition, distribution, or sequence, use a table or chart. Read `references/visualisation.md` before formatting one.
- Label calculated values as `Derived`.
- Do not name skills, MCP tools, or internal tables in the answer.
- Do not suggest follow-up actions unless asked.
