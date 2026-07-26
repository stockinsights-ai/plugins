# Growth and Trends

Use identical metric definitions, statement scope, audit status, and reporting granularity across every compared period.

## Period Interpretation

- Latest/preceding year: fetch annual `last: 2` and use the newest/older value.
- Latest/preceding quarter: fetch quarterly `last: 2`.
- Preceding-year quarter: compare the latest quarter with the same fiscal quarter one year earlier; normally fetch quarterly `last: 5`.
- TTM/last 12 months: sum the latest four standalone quarterly flow values. Do not sum balance-sheet values or cumulative half-year/9M values.
- N-year CAGR: fetch the endpoint from N years earlier plus the current endpoint, requiring N+1 annual observations.

## Formulas

| Requested metric | Formula |
|---|---|
| Absolute change | `current - prior` |
| Growth / YoY / QoQ | `(current / prior - 1) * 100` |
| N-year CAGR | `((ending / beginning)^(1 / N) - 1) * 100` |
| N-period average | `sum(values) / N` |
| N-period median | middle ordered value, or mean of the two middle values |
| Margin expansion/contraction | `current_margin - prior_margin` in percentage points |

Apply these formulas to revenue, operating profit, EBIT, PBT, PAT, EPS, debt, assets, equity, cash flow, or a derived margin when every required input exists.

Do not calculate percentage growth when the prior value is zero. When profit or another endpoint changes sign, report the absolute change and describe it as a turnaround or deterioration instead of presenting a misleading percentage or CAGR. Calculate CAGR only with positive beginning and ending values.

Labels such as `Sales growth`, `Profit growth`, or `Operating cash flow 3years` do not define whether the result is YoY, CAGR, total, or average. Infer the operation only from explicit period wording; otherwise state the chosen interpretation.
