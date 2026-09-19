# Financial Statements

## Purpose and access boundary

Retrieve reported US facts with their period, concept, unit and filing provenance. Read `references/datasets/sql-query-toolkit.md` before SQL. Read `references/datasets/financial-statement-queries.md` for conditional query recipes. Never copy India's metric codes, company joins or crore conversions.

The curated US catalog exposes `public.us_xbrl_income_statements`, `public.us_xbrl_balance_sheets` and `public.us_xbrl_cash_flow_statements`, and the two filing relations that give their rows an issuer. The fact rows are as-reported presentation rows, not a normalized cross-company metric catalog.

## Issuer join

The fact tables have filing_id but no company_id. Each filing_id resolves in one filing relation, joined on `f.id = x.filing_id`:

| Relation | Forms | Use for |
| --- | --- | --- |
| `public.us_10q_filing` | 10-Q, 10-Q/A | Quarterly facts |
| `public.us_20f_filing` | 10-K, 20-F and their amendments, despite the name | Annual facts |

Both carry company_id, which joins to `public.us_companies.company_id`, plus cik, form_type, year, quarter and published_date. Resolve the issuer through `references/datasets/company-data.md`, select its filings by company_id, then read the facts; recipe 1 in `references/datasets/financial-statement-queries.md` is the shape. Never match a filing to an issuer by name text, infer an issuer from a UUID, or use another company's filing because its rows exist.

Extraction covers 10-Q, 10-K and 20-F filings, not amendments, and not every filing is extracted yet. A filing without fact rows is missing extraction, not a zero; read it through `references/datasets/filings-search.md` and `references/datasets/filing-content.md`. A filing_id from another tool is the same identity only when it resolves in one of these relations. A cross-company comparison needs the same concept_qname and unit for every issuer and a bounded list of resolved issuers: an unbounded scan of the fact tables can exceed the statement timeout.

## Shared row schema

Source schema and live database columns agree. Inspect live tool metadata once before use.

| Column | Type | Interpretation |
| --- | --- | --- |
| id | bigint | Storage row key; not amendment precedence |
| filing_id | uuid | Source filing identity; resolves in us_10q_filing or us_20f_filing |
| report_html_file_name | text, nullable | Selected statement report filename |
| row_index | integer | Presentation order; not a metric identifier |
| label | text | Issuer presentation label |
| concept_qname | text | Full tagged concept, including namespace |
| concept_local_name | text | Local tag name; not a curated metric_code |
| component_key | text, nullable | Dimensional member identity from extraction |
| is_total | boolean | True when the extracted row has no component members; not a guarantee it is the desired economic subtotal |
| report_start_date | date, nullable | Flow-period start; null for instant balance-sheet facts |
| report_end_date | date | Flow-period end or balance-sheet date |
| reporting_type | text | Extractor's duration class or instant; see below |
| unit | text, nullable | Normalized reported unit |
| value | numeric(50,18), nullable | Numeric fact; preserve precision and sign |
| created_at, updated_at | timestamptz | Ingestion/update times; not filing acceptance or revision priority |

There are no fiscal_year, fiscal_quarter, company_id, audit_status, statement_scope or metric_code fields in these tables. Report periods and filing provenance must establish those meanings; do not invent columns to satisfy an example.

## Period selection

The extractor classifies duration facts by inclusive day count: quarterly up to 120 days, half_yearly up to 210, nine_months up to 300, annual up to 420. Balance sheets use instant with a null start date. These labels are implementation buckets, not sufficient evidence of a fiscal quarter number. Check exact start/end dates and the filing's stated fiscal period, including 52/53-week years.

One filing may carry current and comparative periods. Select the requested period explicitly. Do not mix quarterly, half_yearly, nine_months and annual facts sharing the same end date. Q1 can also be YTD; a Q2/Q3 cash-flow statement may be cumulative rather than standalone.

- YoY: compare matching duration/scope and corresponding fiscal periods, not arbitrary adjacent rows.
- QoQ: require standalone quarters. Never compare three months with six or nine months.
- Derived quarter: subtract compatible cumulative flows only when concept, currency, scope and revision basis match. Label the result as derived. Never subtract balance-sheet snapshots to call the result revenue or cash flow.
- TTM: four compatible standalone quarters, or compatible annual plus current YTD minus prior comparable YTD. Do not sum overlapping YTD values or infer missing quarters.
- Amendments: choose the requested as-filed or revised basis using authoritative filing metadata. MAX(value), MAX(id) and updated_at do not select the correct revision. If available tools lack revision metadata, state the limitation.

## Concepts, dimensions and duplicates

Start with observed concepts/labels for the selected filing. Match the complete concept_qname and unit, not only a fuzzy label. Issuer extensions are not automatically comparable across companies, even if local names match.

For issuer-wide facts, begin with is_total = true AND component_key IS NULL. This excludes dimensional rows but does not identify revenue, net income or a balance-sheet total by itself. Inspect concept and label as well. The extraction includes members appearing in the selected presentation role; it is not a promise of complete segment disclosure.

A concept can appear at several presentation rows or in several reports. Keep row_index and report_html_file_name until duplicates are understood. Never SUM duplicate presentation occurrences. Identical observations can be collapsed only after their full filing/concept/component/period/unit identity and values agree; conflicting observations require source inspection.

## Units and interpretation

The extractor normalizes currency measures (for example USD), shares, pure and percent, and ratios such as USD_PER_SHARE. Inspect each returned unit. Unit can be null or an extraction fallback; do not infer it from the company's domicile or statement label alone.

Do not apply a displayed table's “in millions” multiplier again without verifying how the fact was stored. Keep raw values until units/scaling are established. Currency facts, share counts and EPS must not share a conversion. Preserve decimal precision; PostgreSQL numerics may arrive as JSON strings. Missing/nil extraction is not zero. A negative value is data; do not silently flip expense or cash-flow signs.

GAAP facts, management-defined adjusted measures, guidance, and analyst estimates are different evidence. Do not relabel one as another. Audit status is not exposed by these rows; do not infer it from ingestion or reporting_type.

## Completion and fallback

For every comparison, report issuer, concept/definition, exact period, unit, derivation (if any) and filing basis. Ratios require compatible numerator and denominator; a zero or missing denominator is undefined. Market-cap ratios additionally require a verified dated market-cap source.

Only use citation links the tools return. SQL rows carry no citation link; preserve provenance in prose (form type, fiscal period, published date) and retrieve a supporting filing through search when a clickable source is needed. Do not fabricate a link from filing_id. If structured issuer retrieval is blocked, explain that limitation and use the filing search and content path. A catalog entry or one successful sample proves neither universal coverage nor production end-to-end readiness.
