---
name: peer-comparison
description: Compare Indian listed companies side by side using financial-statement metrics. Use when the user names companies to compare or asks for a peer, industry, or sector comparison. Resolve the universe, fetch comparable metrics, calculate supported derived metrics, and return a concise Markdown comparison.
---

# Peer Comparison

## Purpose

Compare Indian listed companies using consistent periods, statement scope, units, and metric definitions. Use only data returned by the `stockinsights-in` MCP server.

## Example Queries

- Compare TCS and Infosys on growth, margins, ROE, and cash conversion.
- Compare the largest Indian private-sector banks.
- Show RELIANCE against its closest listed peers.

## Sources and Tools

- Market: `in`
- MCP server: `stockinsights-in`
- Universe tools: `resolve_companies`, `filter_companies`
- Financial tools: `screen_financial_statements`, optionally `list_financial_statement_metrics`

If an MCP tool is unavailable, report the blocker instead of filling values from model memory.

## Retrieval Workflow

1. Determine the universe:
   - Named companies: use clear tickers directly and resolve only ambiguous names.
   - Industry or sector: use `company-data` to map the classification and call `filter_companies`.
   - Peers of one company: fetch its classification, filter on the same four classification levels, exclude the source, and rank candidates by market-cap proximity. Relax `industry_basic`, then `industry`, only when the strict set is too small.
   - Unless the user requests otherwise, compare up to 10 companies for an industry or sector, ordered by market capitalization, and state that selection rule.
2. Honor user-requested metrics. Otherwise use the relevant default profile below.
3. Make one tickers-only `screen_financial_statements` lookup: omit `condition_groups`, include all companies in `tickers`, and request only built-in dependencies in `metrics`.
4. Use no more than 10 built-in metric keys. Prioritize the metrics most relevant to the question when dependencies exceed the limit.
5. Use the same `period`, `statement_scope`, and `audit_status` for every company. Default to consolidated annual figures for the latest two periods so growth and average-balance ratios can be calculated.
6. Calculate derived values only after retrieval and under the `screen-financial-metrics` reference rules.
7. Compare a common fiscal period. If no recent common period exists, show each company's period explicitly and state that they differ.

## Metric Selection

For non-financial companies, prefer the available dependencies for:

- Operating revenue and growth
- Operating profit and margin
- Net profit, growth, and margin
- EPS and ROE
- Total borrowings and debt/equity
- Cash from operations and cash conversion

For banks and NBFCs, prefer:

- Total income and growth
- Net profit, growth, and EPS
- Total assets and equity
- Advances, deposits, and loan-to-deposit ratio
- Total borrowings
- Reported return on assets and gross NPA percentage

Do not apply industrial-company operating-margin, working-capital, liquidity, or ROCE formulas to banks/NBFCs. For mixed industries, prefer directly reported metrics with consistent definitions and separate financial from non-financial companies when sector-specific formulas would mislead.

## Comparison Rules

- Never send a derived name in `metrics`, `condition_groups`, or `sort`; request its built-in dependencies instead.
- Calculate only when every dependency is present and each denominator is non-zero. Never treat missing data as zero.
- Do not mix quarterly flows with annual balance-sheet values.
- Prefer disclosed built-in ratios. Label calculations as `Derived` and state definition-sensitive formulas.
- Use `—` for unavailable values.
- Preserve returned units: amounts are generally INR crore, EPS is INR per share, and ratios/percentages use their natural units.
- Highlight strongest or weakest values only when the preferred direction is unambiguous. Do not declare an overall winner from size or one ratio.
- Do not rank by a derived metric unless every company has a valid value calculated with the same definition and period.

## Response Guidance

Return Markdown only. For five or fewer companies, put metrics in rows. For more than five, put companies in rows.

```markdown
# Peer Comparison — <Companies or Industry/Sector>

**Universe:** <Companies and selection rule>
**Period:** <Common fiscal period, granularity, scope, and audit status>

## Financial Comparison

| Metric | Company A | Company B | Company C |
|---|---:|---:|---:|
| Revenue | ... | ... | ... |
| Revenue growth | ... | ... | ... |

## Key Takeaways

- <Important relative strength or weakness>
- <Growth, profitability, balance-sheet, or cash-flow observation>
- <Comparability or missing-data qualification>

## Methodology Notes

- <Derived definitions, differing periods, or universe caveats when needed>

Source: Internal Database.
```

Omit `Methodology Notes` when no qualification is needed. Do not mention skill or MCP tool names in the final comparison.
