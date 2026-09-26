# Results Calendar

## Purpose

Retrieve the latest expected results dates for Indian listed companies, either market-wide or for named companies.

## Example Queries

- When is TCS expected to report results?
- Show the latest results calendar for INFY and HDFCBANK.
- Which companies are scheduled to report results?

## Data

The latest calendar contains company name, ticker, security code, and expected result date. It supports optional ticker filtering but not arbitrary historical date ranges.

## Sources and Tools

- Market: `in`
- MCP server: `stockinsights-in`
- Primary tool: `get_results_calendar`
- Supporting tool: `resolve_companies` for ambiguous company names

Use only the `stockinsights-in` MCP server. If the tool is unavailable, report the blocker instead of substituting another calendar.

## Input Guidance

Pass confirmed plain tickers, without exchange prefixes, as the tool's comma-separated `ticker` input, for example `"TCS,INFY"`. Omit `ticker` for the latest market-wide calendar.

Do not invent date, period, page, or limit inputs that are not exposed by the current MCP schema.

## Retrieval Workflow

1. Resolve only genuinely ambiguous company identities.
2. Make one `get_results_calendar` call with all requested tickers, or omit the ticker filter for a market-wide request.
3. Report the returned result dates. Do not imply arbitrary historical or date-range filtering is supported.

## Response Guidance

Use `company_name`, `ticker`, `security_code`, and `result_date`. Present dates clearly and preserve the returned timezone/date semantics.

If the response metadata indicates more rows exist than were returned, state that the answer covers the returned latest slice because the current MCP input has no pagination control. If no rows match, name the tickers used and say no scheduled result was returned.

## Validation Checklist

- Uses `get_results_calendar` from `stockinsights-in`.
- Resolves only ambiguous company identities.
- Uses one combined ticker request when filtering.
- Does not claim historical date-range support.
