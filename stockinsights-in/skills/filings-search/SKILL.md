---
name: filings-search
description: Search through company filings - both semantic and keyword search - to answer investor/company related questions.
---

## MCP Server

Use the tools of the `stockinsights-in` MCP server as the data provider.

## Workflow

1. Parse the user intent and resolve ticker(s).
2. Use semantic search, keyword search, or both for any query. The LLM decides the retrieval mix based on what is most likely to produce sufficient evidence; there is no hard query-type restriction.
3. Use latest filings by default. Omit `filing_criteria` when the query does not need a specific filing type or time scope; the API will search latest filings only. Use `time_scope` only when constraining filing type, specific fiscal periods, comparisons across periods, or all historical data.
4. Run one or more retrievals until evidence is sufficient.
5. Answer only from retrieved evidence with inline citations.
6. If evidence is weak or absent, return insufficient data.

## Tools

### `search_filings_semantic`

- Purpose: vector/embedding search across company filing chunks.
- Useful for: strategy, management commentary, risk discussion, outlook, business model, capex plans, competitive positioning, qualitative KPI drivers, or any query where semantic retrieval may find better evidence.
- Supported filing types: `earnings-transcript`, `annual-report`
- Output: JSON response containing relevant chunks plus filing, company, citation title, and `citation_link`.

### `search_filings_keyword`

- Purpose: keyword/full-text search across filing text.
- Useful for: exact phrases, named line items, proper nouns, quoted terms, specific metric wording, validating whether a filing mentions a term, or any query where keyword retrieval may find better evidence.
- Supported filing types: `earnings-transcript`, `annual-report`, `investor-presentation`.
- Output: JSON response containing matched page/chunk text plus filing, company, citation title, `citation_link`, and screenshot links when available. When screenshot links are present, especially for `investor-presentation`, prioritize downloading and analyzing screenshots as primary page-level evidence (as they often contain critical visual tables and charts) and use text fields as a fallback.

## Input contract

Both tools use this request shape:

```json
{
  "query": "search text",
  "filters": {
    "tickers": ["TCS"],
    "filing_criteria": [
      {
        "filing_type": "earnings-transcript",
        "time_scope": {
          "mode": "latest"
        }
      }
    ]
  }
}
```

- `query` is required.
- `filters` may be omitted or set to `null` to search latest filings for all companies and all supported filing types.
- `filters.tickers` may be omitted or set to `null` to search all companies. Use plain tickers like `TCS`, `INFY`, `RELIANCE`.
- `filters.filing_criteria` may be omitted or set to `null`; the API defaults to searching latest filings only for every supported filing type.
- By default, search latest filings because they usually provide the most accurate current answer. Use `all` or `periods` only when the query indicates an alternative time scope.
- Only one criterion is allowed per `filing_type`.

### Time scope

Use `time_scope.mode` to control filing recency:

- `latest`: search only filings marked latest. Omit fiscal period fields. This is the default mode unless the query asks otherwise.
- `all`: search all available history. Do not include fiscal period fields. This mode must not query/filter on the latest column.
- `periods`: search only explicit fiscal periods. This mode is also latest-agnostic; do not include any latest flag.

Period fields:

- `annual-report` uses `fiscal_years`, for example `["FY25", "FY24"]`.
- `earnings-transcript` and `investor-presentation` use `fiscal_quarters`, for example `["FY26Q2", "FY26Q1"]`.
- Omit the incompatible fiscal field.

Examples:

Latest filings for all companies and filing types:

```json
{
  "query": "margin outlook"
}
```

All historical annual reports:

```json
{
  "query": "capital allocation",
  "filters": {
    "filing_criteria": [
      {
        "filing_type": "annual-report",
        "time_scope": { "mode": "all" }
      }
    ]
  }
}
```

Specific quarters for one company:

```json
{
  "query": "deal wins and demand environment",
  "filters": {
    "tickers": ["TCS"],
    "filing_criteria": [
      {
        "filing_type": "earnings-transcript",
        "time_scope": {
          "mode": "periods",
          "fiscal_quarters": ["FY26Q2", "FY26Q1"]
        }
      }
    ]
  }
}
```

### INSTRUCTIONS for selecting filings

**Earnings-transcript** filings contain:

- Guidance (revenue, EBITDA, margins, capex, order book)
- Management outlook or forward-looking commentary
- Segment performance discussion
- Analyst Q&A themes or concerns
- Quarterly performance explanations

**Annual-report** filings contain:

- Long-term strategy
- Annual Audited financial statements
- Business model or segment structure
- Detailed risk disclosures
- Market opportunity or industry positioning

**Quarterly results** released quarterly contain:

- Standardized quarterly unaudited financial performance disclosures
- Income statement, balance sheet, cash flow statement
- Segment-wise financial performance

**Investor-presentations** released quarterly contain:

- Detailed revenue composition, KPIs
- Growth drivers & expansion plans
- Visual breakdowns of business units
- Strategy roadmaps

## Required rules

- Do not use model memory for factual claims.
- Construct payloads from the MCP tool schema for every query.
- Cite every material claim inline.
- Use `citation_link` as the primary citation URL. Include available filing metadata such as document type, year/quarter/date, and company.
- If conflicting or insufficient evidence, say insufficient data and what is missing.
