---
name: filing-content
description: Retrieve the content or source artifacts for exactly one Indian company filing. Use for summaries, detailed analysis, section extraction, or source artifact retrieval for one earnings transcript, annual report, investor presentation, or quarterly result.
---

# Filing Content

## Purpose

Use this skill when the user needs full context from one specific Indian listed-company filing, not a cross-filing search result. It retrieves one filing selected by ticker, filing type, and time scope.

## Example Queries

- Summarize TCS FY25Q3 earnings call.
- Read the latest INFY annual report and summarize the risk disclosures.
- Extract the capex section from RELIANCE FY25 investor presentation.
- Get the quarterly result markdown or financial statement artifacts for HDFCBANK FY26Q1.

## Data

This skill retrieves filing content for the Indian (IN) market.

Supported filing types:

- `earnings-transcript`: quarterly earnings call transcripts with management commentary, guidance, and analyst Q&A.
- `annual-report`: annual audited reports with strategy, audited financials, risk disclosures, and business details.
- `investor-presentation`: quarterly presentations with charts, KPIs, strategy updates, and business breakdowns.
- `quarterly-result`: quarterly result filings with source artifact URLs, such as extracted markdown and financial statement URLs.

Period formats:

- Fiscal quarters use `FYxxQy`, for example `FY25Q3`.
- Fiscal years use `FYxx`, for example `FY25`.

## Sources and Tools

Use the `stockinsights-in` MCP server as the only data provider.

Primary tool:

- `get_filing_content`: retrieves exactly one filing and returns at most one item in `data`.

Supporting tool:

- `resolve_companies`: use this first when the user gives a company name, ambiguous identifier, or exchange-prefixed ticker. Pass a plain Indian ticker such as `TCS` or `RELIANCE` to `get_filing_content`; do not include exchange prefixes.

## Input Guidance

Construct the MCP payload from the current tool schema. The request requires:

- `ticker`: plain Indian ticker, for example `TCS`.
- `filing_type`: one of `earnings-transcript`, `annual-report`, `investor-presentation`, `quarterly-result`.
- `time_scope`: select `latest` or one explicit fiscal period.

Use `time_scope.mode: "latest"` when the user asks for the latest matching filing or does not specify a period:

```json
{
  "ticker": "TCS",
  "filing_type": "earnings-transcript",
  "time_scope": { "mode": "latest" }
}
```

Use `time_scope.mode: "period"` with `fiscal_quarter` for `earnings-transcript`, `investor-presentation`, or `quarterly-result`:

```json
{
  "ticker": "TCS",
  "filing_type": "earnings-transcript",
  "time_scope": {
    "mode": "period",
    "fiscal_quarter": "FY25Q3"
  }
}
```

Use `time_scope.mode: "period"` with `fiscal_year` for `annual-report`:

```json
{
  "ticker": "INFY",
  "filing_type": "annual-report",
  "time_scope": {
    "mode": "period",
    "fiscal_year": "FY25"
  }
}
```

For `latest`, omit `fiscal_year` and `fiscal_quarter`. For `period`, set exactly one compatible fiscal field.

## Retrieval Workflow

1. Parse the user request for company, filing type, and time scope.
2. Resolve the company to a plain ticker when needed.
3. Call `get_filing_content` with the schema-shaped payload.
4. Use the returned filing only; this tool does not return multiple filings.
5. If the filing is not found, has no pages, or has no artifacts, say so clearly and suggest checking the ticker, filing type, or fiscal period.

Use this skill for comprehensive review of one filing. For cross-company or cross-period search, keyword lookup, semantic lookup, announcements, or past N quarters/years across many filings, use the relevant discovery/search skill first and then use this skill only after one filing is selected.

## Response Guidance

The tool returns `status: "success"` and `data`, with at most one filing.

For `content_type: "page_text"`:

- Expected for `earnings-transcript`, `annual-report`, and `investor-presentation`.
- Use `pages[].content` as the filing text.
- Cite material claims inline with `[page N](citation_link)`.
- Use `page_screenshot_url` as supporting visual evidence where useful.
- When summarizing, cover the major sections systematically instead of relying on one excerpt.

For `content_type: "artifact_urls"`:

- Expected for `quarterly-result`.
- Use `artifacts.markdown_url` and `artifacts.financial_statement_urls` as source pointers.
- Do not claim to have read markdown or financial-statement contents unless those URLs are separately fetched and analyzed.
- Cite the filing-level `citation_link` and name the available artifacts.

If evidence is partial, conflicting, or unavailable, state the limitation directly.

## Nuances

- Do not use model memory for factual claims about a filing.
- Do not switch sources when the user asked for StockInsights India MCP-backed filing content.
- Do not use `get_filing_content` for broad search; it selects one filing by public fields.
- The MCP schema uses `time_scope.mode: "period"` for one explicit fiscal period, not `periods`.
- `quarterly-result` returns artifact URLs rather than page text.

## Validation Checklist

- `get_filing_content` was the selected data source.
- Company names or ambiguous identifiers were resolved to a plain ticker.
- Payload used `ticker`, `filing_type`, and `time_scope` only.
- Annual reports used `fiscal_year`; quarterly filing types used `fiscal_quarter`.
- The answer cited page-level evidence for `page_text` results or named artifact URLs for `artifact_urls` results.
- Missing filing/content/artifacts were reported plainly.
