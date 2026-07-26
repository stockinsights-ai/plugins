# Liquidity and Leverage

Use point-in-time balance-sheet values from the same period and statement scope. Balance-sheet data is commonly available at Q2 and Q4; do not treat a missing Q1/Q3 value as zero.

## Formulas

| Requested metric | Formula | Unit |
|---|---|---|
| Working capital | `current_assets - current_liabilities` | INR crore |
| Current ratio | `current_assets / current_liabilities` | ratio |
| Quick ratio | `(current_assets - inventories) / current_liabilities` | ratio |
| Net debt | `total_borrowings - cash_and_equivalents` | INR crore |
| Gross debt/equity | `total_borrowings / total_equity` | ratio |
| Net debt/equity | `net_debt / total_equity` | ratio |
| Debt/assets | `total_borrowings / total_assets` | ratio |
| Equity/assets | `total_equity / total_assets` | ratio |
| Cash/debt | `cash_and_equivalents / total_borrowings` | ratio |
| Current-debt share | `borrowings_current / total_borrowings * 100` | percent |
| Non-current-debt share | `borrowings_noncurrent / total_borrowings * 100` | percent |

Prefer reported `debt_equity_ratio`, `interest_coverage_ratio`, and `debt_service_coverage_ratio` when available. If interest coverage must be derived, use `operating_ebit / finance_costs` and label it derived; it may differ from the disclosed definition. DSCR cannot be derived because principal repayments are unavailable.

Working-capital days is definition-sensitive. If the user accepts total net working capital, calculate `average(current_assets - current_liabilities) / annual_operating_revenue * 365`. Do not equate this with a trade working-capital cycle.

Do not calculate current/quick ratios for banks or NBFCs. Do not derive receivable days, payable days, inventory days, inventory turnover, or cash-conversion cycle because receivables, payables, and COGS are absent.
