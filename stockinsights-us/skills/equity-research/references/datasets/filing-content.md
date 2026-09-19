# Filing Content

## Purpose and identity

Use `get_filing_content` to read part of one US `earnings-transcript`, `10-K`, `10-Q`, or `20-F`. US filings are addressed by chunk, never by PDF page number.

Prefer a search hit's `filing_id` as `document_id`, together with its exact `filing_type`. Do not add `ticker` or `time_scope` to this selector: a later filing must not replace the source you are checking. Example template (replace the placeholder with an actual returned ID):

```json
{"document_id":"<filing_id from search>","filing_type":"10-K","chunk_selection":{"mode":"range","start_chunk":4,"end_chunk":8}}
```

When no document is identified, use `ticker` plus `time_scope`:

```json
{"ticker":"TSLA","filing_type":"earnings-transcript","time_scope":{"mode":"latest"},"chunk_selection":{"mode":"range","start_chunk":1,"end_chunk":5}}
```

```json
{"ticker":"AAPL","filing_type":"10-K","time_scope":{"mode":"period","fiscal_year":"FY24"},"chunk_selection":{"mode":"range","start_chunk":1,"end_chunk":5}}
```

```json
{"ticker":"MSFT","filing_type":"10-Q","time_scope":{"mode":"period","fiscal_quarter":"FY25Q1"},"chunk_selection":{"mode":"range","start_chunk":1,"end_chunk":5}}
```

For period mode, set exactly one fiscal field: 10-K/20-F require `fiscal_year` in FYxx format; transcripts/10-Q require `fiscal_quarter` in FYxxQy format. Do not pass search's `calendar_years` or `fiscal_quarters` arrays here. The period maps to stored year/quarter metadata; verify actual reporting dates in the text because US fiscal calendars differ. These examples demonstrate accepted shapes, not verified content availability.

## Select a region

Copy a semantic search hit's numeric `chunk`; a supplied citation anchor may also identify `#chunkN`. Keyword search does not guarantee a numeric locator. Never use the result's array index or a visible PDF page number as a chunk.

```json
{"mode":"range","start_chunk":4,"end_chunk":8}
```

```json
{"mode":"chunks","chunks":[4,8,12]}
```

Ranges are inclusive and contain at most 10 positive integers, with end at least start. Lists contain 1–10 positive integers; duplicates are removed and results sorted. Do not mix mode-specific fields.

Always request a deliberate small region. An omitted direct API selection returns a bounded opening excerpt, not necessarily ten chunks or the entire filing. The content service spends a 60,000-character budget in whole chunks, always allowing the first chunk; the bound can also shorten an explicitly requested region. The agent's focused wrapper separately defaults to the first ten chunks. Inspect the actual response in either case.

## Response and completeness

The HTTP response has `status` and a `data` array containing a `content_type: "filing_chunks"` object:

- Identity: `company_name`, `ticker`, `filing_type`, `year`, `quarter`, document `citation_link`.
- Addressing: `unit: "chunk"`, `total_chunks`, `truncated`.
- `chunks`: objects with `chunk`, `content`, `citation_link`, `citation_title`, and optional `page_screenshot_url`.
- When selected: `chunk_selection.requested_chunks`, `returned_chunks`, `missing_chunks`.

Read text from `chunks[].content`, not a fabricated `page_text` or `chunk_text` field. Search and content use different text field names.

`truncated` indicates that the character budget omitted fetched rows. It is not a whole-document completeness flag: selecting chunks 4–8 can return `truncated: false` even when the document contains 200 chunks. Compare returned ordinals and `total_chunks`, and state the scope read. `missing_chunks` can reflect nonexistent ordinals or budget omission, so inspect returned content and retry a smaller region before concluding content is absent.

If every requested chunk is absent, the service distinguishes no stored content from an invalid region and can return known bounds in the error. A range past the end does not mean the entire filing is unavailable. Correct a rejected selection once; a 400 is not evidence that the filing is missing.

## Reading workflows

For a specific claim, search first, fetch the hit plus adjacent chunks, and check definitions, qualification, period and units before answering. Expand only when the surrounding passage is incomplete. Keep the original document ID throughout verification.

For an annual report summary, use search to locate business, operating results, liquidity/cash flow, outlook, risks and any material qualifications. Read each relevant region. Opening boilerplate alone cannot support a comprehensive summary. State coverage limits when a requested section cannot be retrieved.

For a transcript, distinguish prepared remarks, analyst questions and management answers. Read adjacent chunks when a question and answer span a boundary. Do not turn a question's assumption into management guidance.

For financial tables, inspect column dates, scale, currency and footnotes; a chunk boundary can separate headers from values. Use `references/datasets/financial-statements.md` for period, unit and comparability rules. Copy source markers supplied by the service; do not generate source numbers or rewrite links.

Report repeated failures honestly without inventing content or switching to another issuer, period or market.
