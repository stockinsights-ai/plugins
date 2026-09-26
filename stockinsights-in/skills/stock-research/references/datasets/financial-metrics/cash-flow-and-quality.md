# Cash Flow and Earnings Quality

Use annual cash flow by default. Half-yearly cash flow is cumulative H1; quarterly and nine-month cash-flow values are generally unavailable.

## Formulas

| Requested metric | Formula | Interpretation |
|---|---|---|
| CFO margin | `cash_from_operations / operating_revenue * 100` | Operating cash generated per unit of revenue. |
| Cash conversion / CFO-to-PAT | `cash_from_operations / net_profit` | Cash backing of accounting earnings. |
| CFO-to-debt | `cash_from_operations / total_borrowings` | Cash-flow debt coverage proxy. |
| Cash return on assets | `cash_from_operations / average(total_assets) * 100` | Cash profitability of assets. |
| Accrual ratio | `(net_profit - cash_from_operations) / average(total_assets) * 100` | Higher positive values imply more accrual-dependent earnings. |
| Net cash flow | `cash_from_operations + cash_from_investing + cash_from_financing` | Prefer built-in `net_change_in_cash` when available. |

For multi-year CFO/CFI/CFF growth, averages, or totals, apply `growth-and-trends.md` only after the requested aggregation is clear. Preserve the reported sign of investing and financing cash flows.

Do not derive free cash flow from `cash_from_operations + cash_from_investing`. Investing cash flow includes acquisitions, investments, disposals, and asset sales; reliable FCF requires a separate capital-expenditure or PPE-purchase metric. Cash at the beginning/end of a cash-flow period is also unavailable.
