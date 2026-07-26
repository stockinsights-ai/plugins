# Profitability and Margins

Use these formulas only for non-financial companies unless the query explicitly defines a bank-specific calculation.

## Operating-Revenue Guard

`revenue` can fall back to `total_income`. For operating formulas, define strict operating revenue as:

1. Confirmed revenue from operations when source provenance is available; otherwise
2. `total_income - other_income` when both values are present.

If neither is possible, do not derive OPM, operating profit, or an operating margin. Do not silently use `total_income` because it includes non-operating income.

## Safe Formulas

| Requested metric or alias | Formula | Unit |
|---|---|---|
| Sales | strict operating revenue | INR crore |
| Operating profit / operating EBITDA | `operating_revenue - total_expenses + finance_costs + depreciation` | INR crore |
| OPM / operating EBITDA margin | `operating_profit / operating_revenue * 100` | percent |
| Operating EBIT | `operating_revenue - total_expenses + finance_costs` | INR crore |
| EBIT margin | `operating_ebit / operating_revenue * 100` | percent |
| PBT margin | `pbt / operating_revenue * 100` | percent |
| NPM / PAT margin | `net_profit / operating_revenue * 100` | percent |
| Effective tax rate | `tax_expense / pbt * 100` | percent |
| Material-cost ratio | `cost_of_materials / operating_revenue * 100` | percent |
| Employee-cost ratio | `employee_cost / operating_revenue * 100` | percent |
| Finance-cost ratio | `finance_costs / operating_revenue * 100` | percent |
| Depreciation ratio | `depreciation / operating_revenue * 100` | percent |
| Other-expense ratio | `other_expenses / operating_revenue * 100` | percent |
| Other-income share | `other_income / total_income * 100` | percent |

Calculate effective tax rate only when PBT is positive and tax expense is not distorted by a material credit or one-off adjustment.

## Conditional Formulas

| Requested metric | Formula | Conditions |
|---|---|---|
| EBIDT | `pbt + finance_costs + depreciation` | Accounting EBIDT, not operating EBITDA; PBT can include other income and exceptional items. |
| Estimated period-end shares | `equity_share_capital / face_value` | Returns crore shares; assumes ordinary fully paid shares with one face value. |
| Book value per share | `total_equity / estimated_shares` | Same scope and period; consolidated BVPS is approximate when equity includes non-controlling interests. |
| Graham number | `sqrt(22.5 * eps_basic * book_value_per_share)` | Calculate only when EPS and book value are positive. |

Do not derive GPM/gross margin because COGS is unavailable. `cost_of_materials` alone is not COGS. Do not treat `exceptional_items` as extraordinary items without confirming the filing definition. Current tax is unavailable separately from total `tax_expense`.
