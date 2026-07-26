# Returns and Efficiency

Prefer annual flow values and the average of opening and closing balance-sheet values. Fetch at least two annual balance-sheet periods. Quarterly annualisation is only an explicitly labelled approximation and can be distorted by seasonality.

## Formulas

| Requested metric | Formula | Status |
|---|---|---|
| ROE | `net_profit / average(total_equity) * 100` | Safe approximation with total equity. Consolidated ROE can be distorted by non-controlling interests; do not mix `net_profit_owners` with total equity. |
| ROA | `net_profit / average(total_assets) * 100` | Safe for non-banks; prefer reported `return_on_assets` for banks/NBFCs. |
| Asset turnover | `operating_revenue / average(total_assets)` | Safe when strict operating revenue is available. |
| Fixed-asset turnover | `operating_revenue / average(property_plant_equipment)` | Safe for asset-based non-financial businesses. |
| Working-capital turnover | `operating_revenue / average(current_assets - current_liabilities)` | Suppress when average working capital is zero, negative, or very small. |
| Financial leverage / equity multiplier | `average(total_assets) / average(total_equity)` | Safe when equity is positive. |

## Definition-Sensitive Formulas

Calculate these only when the answer states the adopted definition:

- ROCE: `operating_ebit / average(total_assets - current_liabilities) * 100`.
- ROIC: `NOPAT / average(total_equity + total_borrowings - cash_and_equivalents) * 100`, where `NOPAT = operating_ebit * (1 - normalized_tax_rate)`.
- Basic earning power: `operating_ebit / average(total_assets) * 100`.

Do not derive ROCE/ROIC when capital employed is negative, the effective tax rate is abnormal, or the company is a bank/NBFC. Different providers use different denominator and tax conventions, so never present these as identical to a disclosed ratio.

Inventory turnover is unavailable because COGS is absent. A complete Piotroski score is unavailable because gross margin, share issuance, and other required inputs are not represented reliably.
