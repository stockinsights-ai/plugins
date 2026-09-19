# Filings Search

## Choose the evidence

Use `search_filings_semantic` for concepts, strategy and management explanations. Use `search_filings_keyword` for exact wording, named products and line items. Both search `earnings-transcript`, `10-K`, `10-Q`, and `20-F`. Transcripts support management discussion and Q&A; annual filings support business, risks and annual statements; 10-Q supports interim disclosures. These tools do not search an 8-K/6-K announcement feed: use `references/datasets/announcement-feed.md` for that.

Route structured statement questions to `references/datasets/financial-statements.md`. Resolve ambiguous names or share classes with `references/datasets/company-data.md` before narrowing the search. US type names are not India's `annual-report` or `investor-presentation`.

## Request contract

Send `query` and `filters`. Query length is 1–500 characters in the published tool contract. A non-null filter contains `tickers` (1–100 plain symbols, or null for all companies), `filing_criteria` (1–4 criteria), and optional `latest` (defaults to true). Each criterion has `filing_type` and `period`.

- Annual historical search: `period: {"calendar_years":["2024"]}`. Use four-digit years, at most 10.
- Quarterly historical search: `period: {"fiscal_quarters":["FY25Q1"]}`. Use FYxxQy, at most 20.
- `period: null` uses `latest`. True selects filings marked latest; false searches available history.
- An explicit period overrides `latest`. Do not send both period field families in one criterion.
- `filters: null` means latest supported filings across all companies. Do not send empty arrays as a wildcard or drop a requested issuer after a failed resolution.

Search uses stored filing year/quarter metadata. A year label is not an assertion about the issuer's fiscal start/end dates. Verify the actual period in the document before making financial comparisons. Search `period` and content `time_scope` are different contracts.

### Latest management discussion

Call `search_filings_semantic`:

```json
{"query":"robotics humanoid robots management outlook","filters":{"tickers":["TSLA"],"filing_criteria":[{"filing_type":"earnings-transcript","period":null}],"latest":true}}
```

For exact product wording, call `search_filings_keyword` with a short query such as `"Optimus" OR "humanoid"`. Keyword syntax supports quoted phrases, OR and minus exclusions; ordinary words narrow the match. Avoid stuffing every synonym into one restrictive query.

### Historical annual disclosure

```json
{"query":"customer concentration supply chain risk","filters":{"tickers":["AAPL"],"filing_criteria":[{"filing_type":"10-K","period":{"calendar_years":["2024"]}}],"latest":false}}
```

### Compare specific quarters across years

```json
{"query":"gross margin outlook","filters":{"tickers":["MSFT"],"filing_criteria":[{"filing_type":"earnings-transcript","period":{"fiscal_quarters":["FY24Q4"]}},{"filing_type":"earnings-transcript","period":{"fiscal_quarters":["FY25Q1"]}}],"latest":false}}
```

Keep cross-year pairs in separate criteria (or calls). Within one criterion the backend flattens years and quarters into separate sets: putting FY24Q4 and FY25Q1 together can also match FY24Q1 and FY25Q4. Multiple criteria of the same type remain separate branches and duplicate hits are removed. For more than four distinct branches, split calls.

These are valid request-shape examples, not claims that the named companies have matching passages.

## Read results correctly

The HTTP envelope contains `status`, `data` and `retrieval`. Hits provide `company_name`, `ticker`, `filing_type`, `filing_id`, `year`, `quarter`, `chunk_text`, `citation_link`, and `citation_title`, with some fields nullable or omitted.

Semantic hits additionally expose `chunk` as a positive integer or null. Keyword hits do not guarantee a numeric chunk locator: do not invent a chunk from their array position. Preserve `filing_id` as the content tool's `document_id`. Use `references/datasets/filing-content.md` to inspect a semantic hit and its neighboring chunks, or run semantic search within the same company/type/period to locate context around a keyword hit. Preserve supplied source markers and citation links.

`retrieval` reports `requested`, `succeeded`, `failed`, `failed_filing_types` and `complete`. Inspect optional `dropped_period_values` and `unresolved_tickers` too. All branches failing produces an error, while partial failure can return useful data with incomplete retrieval. An empty successful response is different from failed retrieval.

`complete: true` means the retrieval branches succeeded; it does not establish exhaustive corpus coverage. Keyword results are capped at 100 across the request. Semantic retrieval asks for the top 20 per branch. Neither exposes pagination for exhausting every match. The unresolved-ticker field identifies the case where none resolve; partial issuer resolution does not guarantee a per-ticker warning. For required multi-company comparisons, verify each requested issuer actually appears or query them individually.

## Research workflow

1. State the company, evidence type and period needed. For “last call,” keep `latest: true` and verify the returned period; do not assume the current calendar quarter has a transcript.
2. When the user named no period, search `latest: true` first, before any historical search. Read the returned `year` and `quarter`: that is the anchor for everything after. Today's date is not evidence a period was filed, and the newest stored filing can sit several quarters behind the calendar.
3. An empty `latest: true` is not an empty corpus. The latest flag is set per filing type and is not always set: an issuer can hold years of a type while no filing of it is marked latest. When the first pass returns nothing, take the anchor from another supported type for the same issuer, or search history explicitly with `latest: false`. Never read the empty result as no coverage.
4. Count back from that anchor when the question needs history, such as a trend, a series or a multi-quarter comparison. Never write a `calendar_years` or `fiscal_quarters` list from the current calendar year, or from an assumption about how far back the data runs. A list that omits the anchor period silently answers a different, older question.
5. Say which period the series ends at, and say so whenever the newest evidence is older than the user would expect. A trend presented without its latest period reads as current.
6. Search a focused concept. When empty, try an exact named term, a less restrictive concept or another appropriate disclosure type while preserving the requested company and period.
7. Read enough surrounding content to distinguish management guidance, analyst questions, reported actuals and historical discussion. A question mentioning a product is not evidence management has adopted it.
8. For a report summary, cover business, financial performance, outlook, risks and relevant Q&A with separate evidence. Search hits alone do not mean the whole report was read.
9. For comparisons, use comparable periods and source types for each issuer. Report missing evidence explicitly instead of treating it as zero exposure or no discussion.

For a thematic screen, a market-wide semantic search can generate candidates. Verify each candidate's operating evidence, separating existing revenue/operations from plans, pilots and adjacent exposure. Then apply supported company or financial filters. Present an evidence-backed shortlist, not an exhaustive sector screen or a ranked revenue exposure table unless comparable revenue evidence was actually found.
