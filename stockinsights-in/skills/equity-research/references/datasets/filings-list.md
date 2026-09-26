# Filings List

Use `list_filings` when the user asks which filings were released recently, or which filings are available for a company and period. The result is filing metadata, ordered by `published_date` descending. It does not search filing text; use `references/datasets/filings-search.md` for passages and `references/datasets/filing-content.md` to read one identified filing.

Specify exactly one `filing_type` per call: `earnings-transcript`, `annual-report`, `investor-presentation`, or `quarterly-result`. For a question spanning types, make one call per type and compare their returned publication dates. Optional `filters.tickers` accepts plain tickers (up to 100); omit it for the market-wide feed. Use `filters.fiscal_years` (FYxx) only for annual reports, or `filters.fiscal_quarters` (FYxxQy) only for the other types. Omit period filters to include every available period. `limit` is 1–50 and defaults to 50.

```json
{"filing_type":"earnings-transcript","filters":{"tickers":["TCS"],"fiscal_quarters":["FY26Q2"]},"limit":10}
```

Each record contains `filing_id`, company identity, filing type, period fields, `citation_link` and `citation_title`; `published_date` may be null. Indian `get_filing_content` does not accept `filing_id`: use the returned ticker, filing type and period with its `time_scope` contract, plus page selection when required. A period label and publication date answer different questions: state both when recency matters. A full first page of 50 results does not establish that no older filings exist. An empty result means no filing matched those filters, not that the company made no disclosures elsewhere. For corporate announcements use `references/datasets/announcement-feed.md`.
