# Financial Data Tables and Visualizations

Use these rules when retrieved evidence contains a meaningful trend, comparison, composition, distribution, or sequence.

## Format Selection

- Use a Markdown table for exact values, direct lookups with multiple fields, rankings, screens, and small comparisons where a chart would add little. Do not force a visual for a single value or a short factual answer.
- When the host supports inline custom visuals, create one when it makes the main relationship materially easier to understand. Do not merely describe the chart or draw it with ASCII characters.
- When custom visuals are unavailable, fall back to a Markdown table. Never omit useful data because the host cannot render a visual.
- Prefer one primary visual per answer. Add another only when it explains a distinct question that the first visual cannot show.

## Visual Selection

- Time series with at least three comparable periods: line chart. Use this for revenue, profit, margins, cash flow, or other trends; label annual versus quarterly periods clearly.
- One-period comparison across at least three companies or metrics: sorted horizontal bar chart.
- Multi-period peer growth: indexed line chart with every company rebased to 100 in the same starting period. State the base period and do not compare indexed values as absolute company size.
- Quarterly observations across multiple years: line or grouped-column chart when seasonality is relevant.
- Segment, product, geographic, cost, asset, or funding composition: stacked bars across periods; use a 100% stacked chart for mix percentages. Use a treemap only for a single-period composition with complete component values.
- Two metrics across at least four companies: scatter plot. Use bubble size for market capitalization only when market-cap data is available and label the size encoding.
- Components that exactly reconcile an opening value to a closing value: waterfall chart. Do not create a waterfall from qualitative drivers or incomplete components.
- Multiple dated corporate events: chronological timeline when sequence matters; otherwise use a table ordered newest first.
- Historical valuation band or price-versus-fundamental chart: use only when the retrieved evidence contains a consistent historical valuation or price series. A current valuation snapshot is not sufficient.

## Safeguards

- Use only retrieved or explicitly calculated values. Never invent missing points, interpolate gaps, or estimate chart coordinates from qualitative statements.
- Keep companies, fiscal periods, reporting granularity, statement scope, audit status, currency, and units comparable. Disclose material differences next to the output.
- Label derived metrics as `Derived` and apply the calculation rules in the financial-metrics references.
- Represent missing values as unavailable; do not convert them to zero.
- Show units on axes and headers. Use plain-English titles that state the metric and period.
- Avoid 3D charts. Avoid radar charts unless the user specifically requests one and every dimension has been normalized with the preferred direction explained.
- Keep citations in the surrounding Markdown because visual marks alone are not sufficient evidence. When exact values matter and are not readable in the visual, include a compact supporting table.
- Follow the visual with one or two concise, cited takeaways. Do not make an unsupported judgment such as declaring an overall winner or treating the upper-right quadrant as inherently best.
