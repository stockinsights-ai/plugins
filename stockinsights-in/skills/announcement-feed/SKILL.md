---
name: announcement-feed
description: Fetch Indian corporate announcements and their summaries. Use for event-driven questions about recent company updates, material developments, management changes, contracts, legal issues, credit rating changes, dividends, disruptions, FDA inspections, earnings-call notices, and other exchange-disclosed market announcements.
---

# Announcement Feed

## Purpose

Use this skill to retrieve Indian corporate announcements disclosed through exchange filings. It supports exact company lookups, market-wide latest announcements, company-universe filters, announcement categories, sentiment/significance filters, and calendar time windows.

Use this for announcement summaries and source links. Do not use it to calculate financial metrics, analyze full filings, or search filing text.

## Example Queries

- Any significant announcements from RELIANCE in the last 7 days?
- Show negative announcements from large-cap banks this month.
- Any management changes at INFY recently?
- Find contract award announcements for L&T and BHEL over the last 3 months.
- Were there any credit rating changes for small-cap pharma companies?
- Show announcements from companies in the Financial Services sector between June 1 and June 27, 2026.
- Any legal disputes or regulatory issues for TATASTEEL recently?
- Give me recent market-wide significant announcements.

## Data

Announcements are event-driven, factual, and immediate disclosures of material developments. The tool returns stored announcement summaries with company metadata and citation links.

Common announcement use cases include:

- corporate actions and restructuring,
- contracts, expansion plans, product launches, and operational disruptions,
- management changes and labor issues,
- legal disputes, regulatory issues, payment defaults, credit rating changes, and US FDA inspections,
- dividends, capital structure changes, earnings reports, investor conferences, and earnings-call notices.

## Sources and Tools

Use the `stockinsights-in` MCP server as the only data provider.

Primary tool:

- `get_announcements`: fetches stored Indian corporate announcement summaries.

## Input Guidance

Construct the MCP payload from the current tool schema. Top-level fields:

- `filters`: announcement filters, or `null` for latest market-wide announcements.
- `limit`: maximum rows to return, from 1 to 50. Defaults to 50. Results are newest first.

`filters: null` or omitted fetches latest announcements across all companies.

Use `filters.categories` for fetching announcements based on categories. Read `references/categories.js` and copy exact category names from that list.

`filters.time_scope: null` or omitted defaults to the latest one-month window. `time_scope.mode: "latest"` also uses the latest one-month window.

Use `time_scope.mode: "last"` with `last: { "value": number, "unit": "days" | "months" }` for relative windows.

Use `time_scope.mode: "range"` with `range: { "from_date": "YYYY-MM-DD", "to_date": "YYYY-MM-DD" }` for inclusive calendar date ranges. Do not use a future `to_date`.

Exact companies:

```json
{
  "filters": {
    "tickers": ["RELIANCE", "INFY"],
    "categories": ["Management Changes"],
    "time_scope": {
      "mode": "last",
      "last": { "value": 7, "unit": "days" }
    }
  },
  "limit": 20
}
```

Market-wide latest announcements:

```json
{
  "filters": null,
  "limit": 50
}
```

Company universe filter:

```json
{
  "filters": {
    "company_filters": {
      "marketcap_categories": ["large"],
      "industry_filters": [{ "industry_sector": "Financial Services" }]
    },
    "sentiment": ["negative"],
    "time_scope": {
      "mode": "last",
      "last": { "value": 1, "unit": "months" }
    }
  },
  "limit": 50
}
```

Explicit date range:

```json
{
  "filters": {
    "categories": ["Management Changes", "Credit Rating Changes"],
    "time_scope": {
      "mode": "range",
      "range": { "from_date": "2026-06-01", "to_date": "2026-06-27" }
    }
  },
  "limit": 25
}
```

## Filter Semantics

- `filters.tickers` accepts plain Indian tickers.
- `filters.company_filters.marketcap_categories` supports `large`, `mid`, `small`, `micro`, and `nano`.
- `filters.company_filters.industry_filters` supports exact `industry_macro`, `industry_sector`, `industry`, and `industry_basic` values. Fields within one object use AND; multiple objects use OR.
  - For exact `industry_filters` values, refer to `company-data/references/industry-classification.json`.
- When both `tickers` and `company_filters` are present, they are intersected.
- `filters.categories` must use exact announcement category names from `references/categories.js`.
- `filters.sentiment` supports `positive`, `negative`, and `neutral`.
- `filters.significance: true` returns significant announcements only; `false` returns non-significant announcements only.

## Retrieval Workflow

1. Parse the user request for company scope, categories, sentiment/significance, and calendar time scope.
2. Build a schema-shaped payload for `get_announcements`.
3. Use `filters.tickers` for exact company questions.
4. Use `filters.company_filters` for broad market-cap or industry screens.
5. Use `filters.categories` to filter announcements based on categories.
6. Use `filters.significance` and `filters.sentiment` when the user asks for significant, negative, positive, or neutral announcements.
7. Call `get_announcements` and answer from the returned summaries and citations.

The current request schema exposes `limit` but no `page` or cursor. If `meta.total_count` is greater than the number of returned rows, state that the answer covers the returned newest slice.

## Response Guidance

The tool returns:

- `status: "success"`
- `data`: announcement rows
- `meta`: `total_count`, `page`, and `limit`

Use `citation_link` for source attribution whenever available. Mention `published_date`, ticker/company, announcement type, and summary text when relevant.

If no rows match, say so plainly and name the filters/time window used.

## Nuances

- Do not use model memory for factual claims about announcements.
- This tool retrieves existing announcement summaries and citation links; it does not read full filing documents.
