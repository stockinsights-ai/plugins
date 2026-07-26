# Filings Search

## MCP Server

Use the tools of the `stockinsights-in` MCP server as the data provider.

## Workflow

1. Route supported financial-statement metrics to the financial-metrics data source. Route a comprehensive summary or section extraction for one identified filing to filing content.
2. For concepts, themes, strategy, guidance, risks, explanations, segment commentary, or non-standard operational KPIs, start with `search_filings_semantic`.
3. For an exact phrase, named project/product, proper noun, or source-native metric label, start with `search_filings_keyword`.
4. For broad `which`, `list`, or `find companies` questions, search across companies without resolving candidates first. For a targeted company question, resolve only ambiguous identities and add ticker and period filters.
5. Use latest filings by default. Omit `filing_criteria` when no filing type or time scope is required; otherwise use `latest`, `all`, or explicit `periods` as requested.
6. Stop when the primary retrieval provides sufficient evidence. If a semantic search is incomplete, run one keyword search using likely source-native labels and useful synonyms; do not repeat the semantic query verbatim.
7. If both searches remain incomplete for one company, retrieve the single most likely investor presentation, annual report, earnings transcript, or quarterly result with `get_filing_content`. Quarterly results are not in either search index and must be retrieved directly.
8. Answer only from retrieved evidence with inline citations. If evidence remains weak, stale, or absent, state what is missing and the periods searched.

## Tools

### `search_filings_semantic`

- Purpose: vector/embedding search across company filing chunks.
- Useful for: strategy, management commentary, risk discussion, outlook, business model, capex plans, competitive positioning, qualitative KPI drivers, or any query where semantic retrieval may find better evidence.
- Supported filing types: `earnings-transcript`, `annual-report`
- Output: JSON response containing relevant chunks plus filing, company, citation title, and `citation_link`.
- Query syntax: use one natural semantic phrase containing the relevant concepts and synonyms. Do not use quotes, `OR`, `AND`, exclusions, or other web-search operators.

### `search_filings_keyword`

- Purpose: keyword/full-text search across filing text.
- Useful for: exact phrases, named line items, proper nouns, quoted terms, specific metric wording, validating whether a filing mentions a term, or any query where keyword retrieval may find better evidence.
- Query syntax: backed by PostgreSQL web-search style full-text search, so the query can use natural language plus boolean-style operators such as `OR`, `AND`, negation with `-term`, and quoted phrases like `"margin expansion"`.
- Supported filing types: `earnings-transcript`, `annual-report`, `investor-presentation`.
- Output: JSON response containing matched page/chunk text plus filing, company, citation title, `citation_link`, and screenshot links when available. When screenshot links are present, especially for `investor-presentation`, prioritize downloading and analyzing screenshots as primary page-level evidence (as they often contain critical visual tables and charts) and use text as fallback.

Keyword-search tips:

- Use quoted phrases for exact wording, for example `"EBITDA per ton"` or `"demand environment"`.
- Use `OR` for alternatives, for example `margin OR profitability`.
- Use `AND` when all concepts should appear, for example `capex AND guidance`.
- Use `-term` to exclude noisy matches, for example `debt -debenture`.
- Keep queries short and evidence-focused. Long natural-language questions can dilute matches; convert them into key terms or phrases.
- Avoid over-constraining with too many `AND` terms, because filings may use synonyms or split related concepts across nearby text.
- Keyword search is lexical, not semantic. Use semantic search as a companion when the filing may discuss the idea without using the exact words.

## Document Routing

- Earnings transcript: guidance, management outlook, segment explanations, and analyst Q&A.
- Annual report: long-term strategy, business model, risks, audited narrative, and market positioning.
- Investor presentation: KPIs, segment/geographic mix, expansion plans, charts, and operating metrics; use keyword search because presentations are not in semantic search.
- Quarterly result/XBRL: use `screen_financial_statements` for supported exact values and `get_filing_content` for source artifacts or page-level context. Never retry semantic or keyword search to find a quarterly result.

## Breakdown and Mix Questions

For segment, geographic, product, or revenue-mix questions:

1. Treat the requested breakdown as the primary retrieval intent, even when its base metric is a supported statement metric.
2. Determine whether the breakdown is statutory or management-defined, then choose the latest source that reports the requested dimensions. Do not hardcode category names.
3. Use `latest` when the user does not specify a period. Search source-native labels and synonyms; keep ticker and period in filters rather than repeating them in the query.
4. If a search result contains the requested values and citation, answer from it. Do not fetch the full filing merely because a page number is available.
5. If evidence is incomplete, follow the workflow fallback to `get_filing_content`. An empty search is no match for that query, not proof that the disclosure does not exist.
6. Do not call an annual-report disclosure the latest reported result unless a newer quarterly source has been checked when relevant. State the source period precisely.

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
