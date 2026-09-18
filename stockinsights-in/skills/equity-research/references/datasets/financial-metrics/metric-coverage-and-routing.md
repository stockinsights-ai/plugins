# Metric Coverage and Routing

Use this index for Screener-style labels whose support is unclear. Historical, annual, quarterly, latest, or preceding suffixes usually select periods rather than define new metrics.

## Direct Built-In Values

- Sales/revenue, total income, other income, expenses, material cost, employee cost, finance cost/interest, depreciation, PBT, tax expense, PAT/net profit, EPS, face value, and exceptional items.
- Debt/borrowings, total assets, total equity, equity capital, reserves, current assets/liabilities, inventory, cash equivalents, and net block/PPE.
- CFO, CFI, CFF, and net change in cash.
- Reported debt/equity, interest coverage, DSCR, bank/NBFC ROA, and gross NPA percentage.

Use period selection for labels such as latest quarter, preceding quarter, preceding-year quarter, last year, preceding year, or N years back. Use the growth reference for growth, averages, medians, CAGR, or TTM.

## Derived from Current Inputs

- Operating profit, OPM, EBIT, EBIT/PBT/PAT margins, expense ratios, current ratio, quick ratio, working capital, net debt, and capital-structure ratios.
- Revenue/profit/EPS/debt/asset growth, historical averages/medians, ROE, ROA, asset turnover, financial leverage, and cash-quality ratios.
- Loan-to-deposit and other bank funding-mix ratios.

Derived values may be calculated in guarded SQL CTEs and used as output, filter, or sort expressions when every row uses the same definition, period, and valid inputs.

## Conditional or Definition-Sensitive

- EBIDT, estimated share count, book value per share, Graham number, ROCE, ROIC, earning power, working-capital turnover/days, and derived interest coverage.
- Use these only with the explicit definitions and guards in the formula references.
- `G Factor` is provider-specific and must not be inferred without its definition.

## Missing Required Statement Inputs

Do not derive:

- GPM/gross margin, inventory turnover, debtor/receivable days, payable days, inventory days, or cash-conversion cycle.
- Free cash flow, dividend, payout ratio, current tax, cash at beginning/end, or capex.
- Gross block, accumulated depreciation, CWIP, investments, lease liabilities, contingent liabilities, trade receivables/payables, customer advances, or preference capital.
- Complete Piotroski score, export percentage, expected/forecast metrics, result dates, or credit ratings.

Use filing search, following `references/datasets/filings-search.md`, for company-reported operational or disclosure metrics.

## Outside Statement-Only Scope

Do not calculate or route through the financial-statement metrics data source:

- Price, market capitalization, PE/PB/PS/PEG, dividend or earnings yield, enterprise value, EV/EBITDA, price/FCF, industry valuation, or historical price returns.
- Volume, DMA, RSI, MACD, highs/lows, or other technical indicators.
- Promoter/public/FII/DII holdings, pledges, shareholder count, SME status, or changes in ownership.

Use the company-data source, following `references/datasets/company-data.md`, only for the market and valuation fields it supports. Do not manufacture unsupported market, ownership, forecast, or technical data.
