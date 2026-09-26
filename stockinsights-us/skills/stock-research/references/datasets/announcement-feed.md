# Announcement Feed

## Purpose

Use `get_announcements` for stored US corporate disclosures, including 8-K/6-K events and their stored AI insights. This is a dated disclosure feed, not press coverage, a full earnings-call transcript or an upcoming earnings calendar. Use `references/datasets/filings-search.md` for searchable periodic filings and transcripts.

## Request contract

Send 1–50 plain company `tickers`. Preserve share-class punctuation and resolve ambiguous entities through `references/datasets/company-data.md`. Multiple tickers use OR.

- `category`: null for all categories, or an array of current category labels.
- `time_period`: null for a rolling last-month default, or `start_date`/`end_date` strings in YYYY-MM-DD format.
- `sentiment`: null or an array of `positive`, `negative`, `neutral` (1–3).
- `significance`: null for either class, true for significant only, false for non-significant only.
- `page`: integer starting at 1; `limit`: 1–50, default 50.

Null filters avoid excluding unclassified or differently classified material. Do not add sentiment/significance filters merely to make the results sound more relevant. The publication window is separate from an event's effective date or a fiscal reporting period.

### Latest stored events

```json
{"tickers":["TSLA"],"category":null,"time_period":null,"sentiment":null,"significance":null,"page":1,"limit":50}
```

### Management changes in a specified window

```json
{"tickers":["AAPL","MSFT"],"category":["Management Changes"],"time_period":{"start_date":"2026-01-01","end_date":"2026-04-01"},"sentiment":null,"significance":null,"page":1,"limit":50}
```

These examples show request shapes and do not claim the named events exist. For a new question, choose the user's dates or inspect the resolved default window. The current implementation converts date strings to instants and queries `published_date > start_date` and `published_date <= end_date`. A date-only end is midnight, so it does not include the rest of that named day. To cover a complete final day, fetch through the following date and inspect/filter boundary results; do not label the raw window as inclusive whole days. The response's `window` states the actual instants searched.

## Supported category labels

Use these exact labels, while treating the currently advertised tool schema as authoritative:

Company Mergers; Disposals and divestitures; Business Restructuring; Expansion Plans; Financial Troubles; Management Changes; Capital Structure Changes; Contract Awards; Legal Disputes; Payment Defaults; Credit Rating Changes; Product Launches; Operational Disruptions; Accounting Changes; Investments/Divestments; Dividend Policy Changes; Labor Issues; Investor Conferences; Earnings Reports; Delisting Actions; IPO Launches; Name Changes; Offer for Sale; US FDA Inspections; Earnings Calls; Other Situations.

Category labels are classification buckets, not promises that every disclosure has a complete or accurate label. For an important negative finding, retry with category null in the same issuer/window and inspect returned summaries before stating that no matching event was found.

## Response and pagination

The HTTP envelope contains `status`, `data`, `meta`, and `window`. `meta` has `total_count`, `page` and `limit`. The `has_more` flag is top-level, not inside `meta`. `window` has resolved `start_date`, `end_date` and `source` (`requested` or the default-window label).

Each data item can include company identity, `type`, `year`, `published_date`, `source_link`, `company_page_url`, `citation_link`, `citation_title` and `ai_insights`. These are disclosure records with stored insights; they are not transcript chunks. Inspect the actual insight fields and preserve supplied source markers. Attribute a material claim to its disclosure and date, distinguishing stored interpretation from quoted source text.

For further pages, increment `page` and preserve every other filter and the explicit date window. With `time_period: null`, the server recomputes the rolling window on each call; use a fixed requested window for a reproducible multi-page review. `total_count` counts matching database records, not necessarily successfully parsed data items. Optional top-level `unreadable_rows` reports dropped rows; therefore a short page is not proof that results are exhausted. Stop when `has_more` is false, or state the number/pages inspected if stopping early.

If none of the tickers resolve, the response is empty with `unresolved_tickers` and count zero; `has_more` may be absent on this path. Never widen the company scope to compensate. Partial ticker resolution does not guarantee individual unresolved warnings, so verify coverage separately for each issuer in a comparison.

## Interpretation and fallback

Group related disclosures carefully to avoid counting an original filing and a follow-up as independent events. Preserve publication dates and distinguish them from announcement effective dates. Sentiment and significance are stored classifications, not facts established by the filing, nor investment recommendations.

No results means no matching records were returned for the chosen issuer, filters and publication window. It does not prove no event occurred. Check identity, resolved dates, paging and classification filters before making a negative statement; report unreadable rows or missing coverage. Say what was searched and that no matching disclosure was found; do not fill the gap from memory.

Future earnings dates, ownership datasets and normalized segment results are not supplied by this tool. Do not substitute India tools.
