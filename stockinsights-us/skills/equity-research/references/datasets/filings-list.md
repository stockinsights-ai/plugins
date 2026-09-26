# Filings List

Use `list_filings` to see which US filings were published recently, or which filings are available for a company and fiscal period. Results are filing metadata ordered by `published_date` descending. This tool does not search content; use `references/datasets/filings-search.md` for passages and `references/datasets/filing-content.md` to read one identified filing.

Send one required `filing_type` per call: `earnings-transcript`, `10-K`, `10-Q`, or `20-F`. For a question spanning types, call each type separately and compare publication dates. Optional `filters.tickers` accepts up to 100 plain tickers; omit it for the market-wide feed. Use `filters.fiscal_years` (FYxx) only for 10-K/20-F, or `filters.fiscal_quarters` (FYxxQy) only for transcripts/10-Q. Omit period filters for all available periods. `limit` is 1–50 and defaults to 50.

```json
{"filing_type":"10-Q","filters":{"tickers":["MSFT"],"fiscal_quarters":["FY25Q2"]},"limit":10}
```

Each record includes `filing_id`, company identity, filing type, period fields, `citation_link` and `citation_title`; `published_date` may be null. Pass the returned `filing_id` as `document_id` with the same `filing_type` to `get_filing_content` so a newer filing cannot replace it. Verify the issuer's fiscal period in the filing before comparing financial facts. A full first page of 50 results does not prove older filings are absent. An empty result only means no filing matched these filters. For 8-K/6-K disclosures use `references/datasets/announcement-feed.md`.
