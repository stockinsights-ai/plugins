# US Statement Query Recipes

Read `references/datasets/financial-statements.md` first. Every recipe starts from a resolved issuer's company_id and reaches facts through that issuer's filings. Never use an arbitrary filing to answer a company question.

## Worked sample and adaptation

The literals below are database validation fixtures, not defaults: company_id 'aCkPWrh3F2' is Tesla, Inc. (TSLA), and filing_id '069f8962-be91-7201-8000-2cce3c54c24d' is its 10-K for fiscal 2025. The queries were executed read-only to verify the schema and result shape. Before using a recipe for a user question, replace each literal with the identity established by `references/datasets/company-data.md` and recipe 1, inspect the relevant relation, and verify periods and concepts from that filing. Do not copy a fixture into another company's answer.

### 1. List the issuer's filings that carry facts

```sql
SELECT f.form_type, f.id AS filing_id, f.year, f.quarter, f.published_date
FROM (
  SELECT id, company_id, form_type, year, quarter, published_date FROM public.us_10q_filing
  UNION ALL
  SELECT id, company_id, form_type, year, quarter, published_date FROM public.us_20f_filing
) AS f
WHERE f.company_id = 'aCkPWrh3F2'
  AND EXISTS (SELECT 1 FROM public.us_xbrl_income_statements AS x WHERE x.filing_id = f.id)
ORDER BY f.published_date DESC, f.id
LIMIT 8
```

Quarterly filings come from us_10q_filing; annual 10-K and 20-F filings come from us_20f_filing. year and quarter are stored filing labels; the fact dates establish the period. For a balance-sheet or cash-flow question, test that relation in the EXISTS clause instead. A filing with no fact rows has not been extracted; read it through `references/datasets/filing-content.md` rather than report zero.

### 2. Discover available periods and units within the selected filing

```sql
SELECT report_start_date, report_end_date, reporting_type, unit,
       COUNT(*) AS presentation_rows
FROM public.us_xbrl_income_statements
WHERE filing_id = '069f8962-be91-7201-8000-2cce3c54c24d'::uuid
  AND is_total = true AND component_key IS NULL
GROUP BY report_start_date, report_end_date, reporting_type, unit
ORDER BY report_end_date DESC, report_start_date, reporting_type, unit
LIMIT 50
```

presentation_rows counts stored presentation occurrences, not distinct economic metrics. Select the period the user requested; latest report_end_date is not automatically the desired fiscal year or revision. Do not combine unit groups.

### 3. Inspect concepts before choosing a metric

```sql
SELECT concept_qname, concept_local_name, label, unit,
       report_html_file_name, row_index, report_start_date,
       report_end_date, reporting_type
FROM public.us_xbrl_income_statements
WHERE filing_id = '069f8962-be91-7201-8000-2cce3c54c24d'::uuid
  AND is_total = true AND component_key IS NULL
ORDER BY report_end_date DESC, report_start_date, row_index, id
LIMIT 50
```

A truncated or capped response is a partial concept inventory. Narrow by the requested statement/period; do not infer that omitted concepts are unavailable. An issuer's label and full QName establish the candidate meaning. No universal revenue or earnings fallback hierarchy has been verified for this dataset.

### 4. Retrieve an observed concept with provenance

This fixture contains the observed concept us-gaap:EarningsPerShareDiluted with unit USD_PER_SHARE. That observation is not a universal coverage promise.

```sql
SELECT filing_id, report_html_file_name, row_index, label, concept_qname,
       report_start_date, report_end_date, reporting_type, unit, value
FROM public.us_xbrl_income_statements
WHERE filing_id = '069f8962-be91-7201-8000-2cce3c54c24d'::uuid
  AND concept_qname = 'us-gaap:EarningsPerShareDiluted'
  AND unit = 'USD_PER_SHARE'
  AND is_total = true AND component_key IS NULL
ORDER BY report_end_date DESC, report_start_date, row_index, id
LIMIT 20
```

This intentionally retains all observed periods for the single concept so the requested period can be selected without guessing dates. Do not sum EPS, average diluted and basic EPS, or convert EPS to millions. Preserve the returned sign and distinguish reported from adjusted EPS.

### 5. Follow one concept across the issuer's annual filings

```sql
SELECT f.form_type, f.id AS filing_id, f.published_date,
       x.report_start_date, x.report_end_date, x.reporting_type, x.unit, x.value
FROM public.us_20f_filing AS f
JOIN public.us_xbrl_income_statements AS x ON x.filing_id = f.id
WHERE f.company_id = 'aCkPWrh3F2'
  AND x.concept_qname = 'us-gaap:EarningsPerShareDiluted'
  AND x.unit = 'USD_PER_SHARE'
  AND x.reporting_type = 'annual'
  AND x.is_total = true AND x.component_key IS NULL
ORDER BY f.published_date DESC, x.report_end_date DESC, x.id
LIMIT 20
```

Each annual filing carries prior years as comparatives, so one fiscal year can appear in several filings. Keep form_type, filing_id and published_date, choose one observation per period deliberately (normally the latest filing's value, stated as such), and never sum or average the repeats. For quarters, use us_10q_filing and reporting_type = 'quarterly'; the fourth quarter is not filed on a 10-Q.

## Adapting to balance sheets and cash flow

The three fact relations share columns and the same filing join. For balance sheets, inspect the selected filing's balance-sheet concepts and use reporting_type = 'instant' with the requested report_end_date; report_start_date should be null. For cash flow, retain both date boundaries and select annual versus cumulative/standalone flow explicitly. Reinspect actual units and concepts rather than carrying over the income-statement predicate.

If zero rows return, check company_id, filing identity, selected relation, period, concept and unit against the returned results. Do not drop the filing or company predicate or change missing data to zero. If the issuer has no extracted filing for the period, use the filing-content fallback.

## Computing a comparison

Compute only after selecting one defensible observation per requested period and scope. For a positive nonzero comparable prior value, growth percent is 100 × (current / prior − 1). If the base is zero, negative, missing or changes definition, explain why an ordinary growth percentage is misleading or undefined and report the absolute change with context. Distinguish a derived calculation from a reported fact and retain both source observations.
