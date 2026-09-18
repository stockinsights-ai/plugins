# Filing Content

## Purpose

Use this reference when the user needs page-level context from one specific Indian listed-company filing, not a cross-filing search result. It retrieves one filing selected by ticker, filing type, and time scope. Annual reports cannot be retrieved in full: locate relevant pages with filing search, then request those pages or a surrounding page range.

## Example Queries

- Summarize TCS FY25Q3 earnings call.
- Read the pages around INFY's latest annual-report disclosure about cybersecurity risks.
- Extract the capex section from RELIANCE FY25 investor presentation.
- Get the quarterly result markdown or financial statement artifacts for HDFCBANK FY26Q1.

## Data

This data source retrieves filing content for the Indian (IN) market.

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

- `get_filing_content`: retrieves exactly one filing and returns at most one item in `data`. It can select an inclusive page range or specific page numbers, with at most 10 pages per request.

Supporting tool:

- `resolve_companies`: use this first when the user gives a company name, ambiguous identifier, or exchange-prefixed ticker. Pass a plain Indian ticker such as `TCS` or `RELIANCE` to `get_filing_content`; do not include exchange prefixes.

## Input Guidance

Construct the MCP payload from the current tool schema. Every request requires:

- `ticker`: plain Indian ticker, for example `TCS`.
- `filing_type`: one of `earnings-transcript`, `annual-report`, `investor-presentation`, `quarterly-result`.
- `time_scope`: select `latest` or one explicit fiscal period.

`page_selection` depends on the filing type:

- It is required for `annual-report` because the tool does not return an entire annual report.
- It is optional for `earnings-transcript` and `investor-presentation`. Prefer it whenever the relevant page numbers are known so unrelated filing text does not consume context.
- It is unsupported for `quarterly-result`; omit it.

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
  },
  "page_selection": {
    "mode": "range",
    "start_page": 72,
    "end_page": 79
  }
}
```

For `latest`, omit `fiscal_year` and `fiscal_quarter`. For `period`, set exactly one compatible fiscal field.

### Page selection

Use `mode: "range"` for one contiguous, inclusive page interval. Set only `start_page` and `end_page`; both must be positive, `end_page` must be at least `start_page`, and the interval can contain at most 10 pages:

```json
{
  "page_selection": {
    "mode": "range",
    "start_page": 10,
    "end_page": 19
  }
}
```

Use `mode: "pages"` for 1 to 10 specific, potentially non-contiguous positive page numbers. Set only `pages`. Avoid duplicates even though the backend deduplicates them:

```json
{
  "page_selection": {
    "mode": "pages",
    "pages": [12, 27, 43]
  }
}
```

Do not combine range fields with `pages` or request more than 10 pages.

## Retrieval Workflow

1. Parse the user request for company, filing type, and time scope.
2. Resolve the company to a plain ticker when needed.
3. For an annual report, use filing search to locate relevant result pages unless the user already supplied page numbers. Read `references/datasets/filings-search.md` and use the result's page or citation metadata; do not attempt to retrieve the whole report.
4. Whenever the user, a search result, or prior retrieved content identifies relevant pages, request up to 10 of them with `page_selection` instead of retrieving the full filing. Use a range to include nearby context around one search hit, or `pages` to retrieve non-contiguous pages identified by one or more results. Keep every page number positive and the selection within the filing.
5. Call `get_filing_content` with the schema-shaped payload. For a transcript or presentation, omit `page_selection` only when the task genuinely requires a comprehensive review and no useful page target is known. For a quarterly result, always omit it.
6. Use the returned filing only; this tool does not return multiple filings. Make another targeted request only when the returned pages show that necessary context continues elsewhere.
7. If the filing is not found, requested pages are unavailable, or artifacts are missing, say so clearly and suggest checking the ticker, filing type, fiscal period, or page selection.

Use this data source for comprehensive review of a transcript or presentation, and for targeted page review of an annual report. For cross-company or cross-period search, keyword lookup, semantic lookup, announcements, or past N quarters/years across many filings, use filing search first and retrieve filing content only after one filing is selected.

## Response Guidance

The tool returns `status: "success"` and `data`, with at most one filing.

For `content_type: "page_text"`:

- Expected for `earnings-transcript`, `annual-report`, and `investor-presentation`.
- Use `pages[].content` as the filing text.
- Follow the system citation rules for every material claim drawn from a page.
- A page carrying `page_screenshot_url` arrives already read from that image, so `pages[].content` is the page's own table rather than its figures flattened. Investor presentations are the filing type this applies to. Quote such a table as it stands, and never compute a figure it does not state — a total that is missing from the page is missing from the filing.
- When summarizing a complete transcript or presentation, cover the major sections systematically instead of relying on one excerpt. For an annual report, limit conclusions to the selected pages and do not imply that they represent the whole report.

For `content_type: "artifact_urls"`:

- Expected for `quarterly-result`.
- Use `artifacts.markdown_url` and `artifacts.financial_statement_urls` as source pointers. Fetch the URL content with the host's URL-fetch tool when one is available; otherwise cite the artifact links without claiming their contents.
- Do not claim to have read markdown or financial-statement contents unless those URLs are separately fetched and analyzed.
- Name the available artifacts without claiming to have read them, and follow the system citation rules when the returned record is citable.

If evidence is partial, conflicting, or unavailable, state the limitation directly.

## Nuances

- Do not use model memory for factual claims about a filing.
- Do not switch sources when the user asked for StockInsights India MCP-backed filing content.
- Do not use `get_filing_content` for broad search; it selects one filing by public fields.
- The MCP schema uses `time_scope.mode: "period"` for one explicit fiscal period, not `periods`.
- Annual reports require `page_selection`; search for relevant pages first when the user did not supply them.
- A page selection returns at most 10 pages. Use either an inclusive range or specific page numbers, never both.
- `page_selection` is optional for earnings transcripts and investor presentations, but preferred when relevant pages are known to avoid filling the context with unrelated content.
- `quarterly-result` returns artifact URLs rather than page text.
- Do not send `page_selection` for `quarterly-result`.

## Validation Checklist

- `get_filing_content` was the selected data source.
- Company names or ambiguous identifiers were resolved to a plain ticker.
- Payload used `ticker`, `filing_type`, `time_scope`, and only a compatible `page_selection` when applicable.
- Annual reports used `fiscal_year`; quarterly filing types used `fiscal_quarter`.
- Annual reports used filing search or user-supplied page numbers followed by a page selection of at most 10 pages.
- Page selection used either valid inclusive range fields or 1 to 10 specific positive page numbers, not both.
- Known relevant pages were retrieved with `page_selection` rather than by loading the full transcript or presentation.
- Material claims followed the system citation rules; `artifact_urls` results named only the available artifacts.
- Missing filing/content/artifacts were reported plainly.
