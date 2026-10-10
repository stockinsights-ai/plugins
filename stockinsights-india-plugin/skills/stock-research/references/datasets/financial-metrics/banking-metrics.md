# Banking and NBFC Metrics

Use bank/NBFC-specific statement fields and disclosed ratios. Do not apply industrial-company OPM, EBITDA, current ratio, quick ratio, working-capital, ROCE, or inventory formulas.

## Available and Derivable Metrics

| Requested metric | Source or formula | Status |
|---|---|---|
| Return on assets | `return_on_assets` | Prefer the disclosed built-in ratio. |
| Gross NPA percentage | `gross_npa_pct` | Direct disclosed metric. |
| Advances / loans | `advances` | Direct balance-sheet metric. |
| Deposits | `deposits` | Direct balance-sheet metric. |
| Loan-to-deposit ratio | `advances / deposits` | Safe when both are from the same period and scope. |
| Advances/assets | `advances / total_assets * 100` | Funding/deployment mix. |
| Deposits/assets | `deposits / total_assets * 100` | Funding mix. |
| Borrowings/deposits | `total_borrowings / deposits * 100` | Wholesale-funding reliance proxy. |
| Credit growth | Apply YoY/CAGR to `advances` | Use `growth-and-trends.md`. |
| Deposit growth | Apply YoY/CAGR to `deposits` | Use `growth-and-trends.md`. |

Do not derive NIM, net NPA, provision-coverage ratio, credit cost, CASA ratio, capital-adequacy ratio, slippage ratio, or cost-to-income ratio. Their required banking line items are not in the current metric catalog.
