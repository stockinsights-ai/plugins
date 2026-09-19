# stockinsights.ai US

Evidence-backed equity research on US listed companies, powered by the stockinsights.ai US MCP server.

## Setup

Install the plugin. On first use, the `stockinsights-us` MCP server (`https://api-us.stockinsights.ai/mcp`) opens a browser window for stockinsights.ai sign-in (OAuth). If tools are unavailable later, re-authorise the connector.

## Skills

| Skill | Use |
| --- | --- |
| `equity-research` | Answers questions about US listed companies. It covers reported financial statements, company profiles and peers, SEC filings and earnings calls, and corporate disclosures. Every figure is cited to its source, with currency, units and the issuer's fiscal period stated. |
| `setup` | Verifies the MCP connection and OAuth sign-in. |

The research skill triggers automatically on questions such as:

- "What was Apple's revenue and diluted EPS over the last 3 fiscal years?"
- "Who are Tesla's industry peers?"
- "What did NVIDIA management say about data-center demand last quarter?"
- "Latest 8-K disclosures for Boeing"

## Data covered

- Company profiles, industry classification, and a stored market-cap snapshot
- XBRL income statements, balance sheets, and cash flows from 10-K, 10-Q, and 20-F filings
- Earnings-call transcripts, 10-K, 10-Q, and 20-F filings (search and read)
- 8-K/6-K corporate disclosures

Not available: live prices, brokerage research, shareholding data, and a US earnings calendar.
