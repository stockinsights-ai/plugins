---
name: company-data
description: The skill contains instructions to resolve company names/tickers and fetch company details or filter companies. Company details include company profile, industry, sector, marketdata such as marketcap, current price, 52 week range, and PE ratio. Filter the companies based on marketcap, industry, sector, PE ratio etc. Use this skill when the a query requires company details, market data, or discover peer/related companies.
examples:
  - What is the market cap and sector of Reliance Industries?
  - Find mid cap auto companies in India
  - Show peers of TCS in the IT services industry
  - List companies in the pharmaceuticals sector with PE below 25
---

## Purpose

Use this skill to resolve company names/tickers and answer structured company profile and market data queries, screen companies, and discover peers.

## MCP Server

Use the tools of `stockinsights-in` mcp server as the data provider

## Company Resolution

Use the `resolve_companies` tool to resolve company names/identifiers. Resolve one or more company names or partial identifiers into confirmed ticker symbols and company details. The tool returns up to 5 candidate matches per entity, ranked by confidence score.

### When to use

- Tickers are not known yet and the user mentions company names (e.g. "ICICI Bank", "Zomato", "Reliance")
- A company name is ambiguous and needs disambiguation before calling other tools
- You want to confirm the correct ticker before running embeddings-search, keyword-search, announcements, or quarterly-results-markdown

## Company filtering

Use the `filter_companies` tool to fetch companies based on resolved ticker or filter companies based on marketcap, industry, sector, PE ratio etc. This tool can be used to discover peers or related companies.

### Filters

The `filter_companies` tool supports following filters:

- Tickers
- Industry attributes
- Market-cap categories
- Market cap, current price, and PE ratio comparisons
- Sorting by company name, ticker, marketcap, price, or PE ratio
- Cursor pagination with a maximum of 50 rows per call

If the query does not specify an order, use the default `marketcap desc`. Keep the same `sort` object when following `meta.next_cursor`.

### Industry classification data reference

Read `references/industry-classification.json` when you want to filter companies based on industry terms such as auto, banking, IT, or pharma.

- Copy exact field names and values from one or more matching rows.
- Do not normalize capitalization, punctuation, or legacy variants.
- Fields within one selected row use AND; multiple selected rows use OR.
- Ask for clarification when the matching rows represent materially different businesses.

### Finding peers of a company

1. Resolve the source company using the `resolve_companies` tool.
2. Fetch its company details using the existing exact company-data query.
3. Call the filter API using all four returned industry classification levels.
4. If too few peers are returned, omit `industry_basic`, then `industry`, then `industry_sector`.
5. Exclude the source company and sort the remaining companies by market-cap proximity.

### Workflow

1. Resolve company names or ambiguous tickers with the `resolve_companies` tool.
2. Based on the query, call the `filter_companies` tool by filling appropriate values for filters, sorting and provide cursor when more results are required.

## Output rules

- Keep the final answer concise.
- If no rows match, say so plainly.
