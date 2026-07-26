---
name: equity-research-workflow
description: Default workflow for research on Indian listed companies using stockinsights.ai. Covers company information, financial statements, screening, filings, announcements, results dates, qualitative research, and direct factual questions.
---

# Equity Research Workflow

## Purpose

Answer Indian equity-research questions in grounded-evidence mode using the `stockinsights-in` MCP server. This is the default workflow when no more specific outcome workflow applies.

Before retrieving data, choose one primary source from the routing table. Do not call several tools merely to discover which one works. Read only the references required for the selected path and any documented fallback.

## Dataset References

- Company resolution, profiles, market data, valuation snapshots, company filters, and peer discovery: `../references/datasets/company-data.md`
- Financial-statement metrics, comparisons, screens, and statement-derived calculations: `../references/datasets/screen-financial-metrics.md`
- Semantic and keyword search across filings: `../references/datasets/filings-search.md`
- Full content or source artifacts for exactly one filing: `../references/datasets/filing-content.md`
- Corporate announcements and recent material events: `../references/datasets/announcement-feed.md`
- Expected company results dates: `../references/datasets/results-calendar.md`

Supporting reference data:

- Exact industry classifications: `../references/datasets/industry-classification.json`
- Exact announcement categories: `../references/datasets/announcement-categories.js`
- Financial calculation and metric-routing rules: `../references/datasets/financial-metrics/`

## Retrieval Routing

Choose the primary source before the first MCP call:

- Financial-statement metrics such as revenue, profit, expenses, debt, assets, cash flow, EPS, solvency ratios, and statement-derived ratios: read `screen-financial-metrics.md` and use `screen_financial_statements` first.
- Market and valuation fields such as market cap, current price, 52-week range, PE, sector, industry, or direct peer discovery: read `company-data.md` and use the company tools.
- Operational or company-defined KPIs such as order book, capacity, utilization, ARPU, volumes, segment mix, or management-defined metrics: read `filings-search.md` and start with semantic search.
- Broad qualitative or cross-company questions about business activity, investment, strategy, guidance, risks, or disclosures: read `filings-search.md` and start with semantic search across the requested universe.
- Exact phrases, named projects/products, proper nouns, or source-native metric labels: read `filings-search.md` and use keyword search first.
- Breakdown, mix, or composition questions: treat the requested breakdown as the primary intent even when the base metric is a supported statement metric. Use the latest source that reports the requested dimensions; do not assume category names or document type.
- Recent material events such as contracts, acquisitions, expansions, regulatory actions, management changes, dividends, or credit-rating changes: read `announcement-feed.md` and use announcements.
- A comprehensive summary, detailed analysis, or section extraction for exactly one identified filing: read `filing-content.md` and retrieve that filing.
- A latest expected results-date or calendar question: read `results-calendar.md` and use the results calendar.
- Use company resolution only when identity is genuinely ambiguous or a confirmed ticker is required. Do not resolve obvious tickers or use resolution for broad candidate discovery.

## Search and Filing Fallback

For a company-focused filing question:

1. Start with semantic search using ticker and period filters.
2. If evidence is missing or incomplete, run one keyword search using likely source-native labels and useful synonyms.
3. Treat evidence as insufficient when the requested information is missing, incomplete, or stale relative to a newer reporting period visible in the results.
4. If both searches are insufficient, retrieve the most likely filing with `get_filing_content`. For a latest exact data point, prefer the latest investor presentation or quarterly result and stop when one source provides complete evidence.

Semantic search covers earnings transcripts and annual reports. Keyword search also covers investor presentations. Quarterly results are not available through either search path and must be retrieved directly with `get_filing_content`.

## Period Selection

- If the user specifies a period, use exactly that period.
- If the user asks for the latest data or omits the period, use the selected tool schema's `latest` mode.
- Calendar context does not prove that a filing exists. Never convert the current calendar quarter or year into an explicit filing period; let the data source select the latest available reported period.
- If an explicitly requested period is unavailable, say so. Do not silently substitute another period.
- Do not substitute a consolidated value, another period, or another proxy for a requested breakdown unless the user asks for it.

## Tool and Evidence Rules

- Use only the `stockinsights-in` MCP server for financial evidence. Do not use web search or model memory for financial claims.
- Construct every payload from the current MCP tool schema. Do not invent fields.
- Use plain Indian tickers without a leading `$` when tools require tickers. Resolve names only when necessary.
- Use the minimum calls needed to answer the question. Follow additional calls only when the selected reference documents a fallback or the returned evidence is incomplete.
- If the requested source or tool is unavailable, report the blocker instead of silently switching sources.
- Every material factual claim must have an inline citation.
- Prefer the returned `citation_link` and include relevant company, document type, fiscal period, or publication date in the citation label.
- If no URL is available for an internal dataset, identify it as `Internal Database` inline.
- If evidence is weak, conflicting, incomplete, or unavailable, state what is missing.

## Response Rules

- Return valid Markdown with a direct, concise answer.
- Prefer a table or visualization over prose-only presentation whenever the evidence contains a meaningful trend, comparison, composition, distribution, or sequence.
- When a table or visualization may help, read and follow `../references/visualisations/financial-data-visualisation.md` before formatting the answer.
- Put citations immediately after the claims they support.
- Format multiple citations as adjacent links, not a parenthesized or comma-separated bundle.
- Never expose internal citation tokens or IDs.
- Do not mention skill, MCP tool, or internal table names in the final answer.
- Do not suggest follow-up actions unless the user asks for them.
