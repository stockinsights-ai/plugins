# Company Data

## Purpose and retrieval order

Resolve US issuers, retrieve profiles, and construct explicit industry or peer universes. Use `resolve_companies` for identity; use `query_structured_financial_data` for structured filters after reading `references/datasets/sql-query-toolkit.md`. Company resolution is not an industry screen.

1. Resolve names or ambiguous symbols. Preserve returned company_id, ticker punctuation and zero-padded CIK; do not construct identifiers or add exchange prefixes.
2. Confirm the intended issuer/share class. If resolution fails, report missing coverage; never silently widen to all companies.
3. Inspect `public.us_companies` once before SQL. Query only the fields needed for the question.
4. For thematic peers, retrieve evidence through `references/datasets/filings-search.md` first, then enrich the resulting issuers. Separate industry membership from demonstrated business exposure.

## Company master

Source schema and a read-only database inspection agree on these fields. Live tool metadata remains authoritative if deployment changes.

| Column | Type | Meaning and use |
| --- | --- | --- |
| company_id | text, unique, non-null | Application issuer identity; distinct from the row UUID and CIK |
| id | uuid, primary key | Company row identity; not a filing identifier |
| company_name | text, non-null | Stored display/legal name |
| stock_ticker | text, nullable | Primary stored symbol; preserve punctuation |
| cik | text, nullable | SEC issuer identifier; retain leading zeros |
| category | text, nullable | Source category; discover actual values before filtering |
| is_delisted | boolean, non-null | Exclude true for a current listed universe; retain for historical questions |
| industry_sector, industry | text, nullable | Source classifications, not India's four-level hierarchy |
| sic, sic_sector, sic_industry | text, nullable | Separate SIC classification; do not interchange with industry fields |
| marketcap | bigint, nullable | Stored market-cap snapshot; unit/freshness limitations below |
| marketcap_category | text, nullable | Stored category; not a substitute for a verified numerical threshold |
| exchange_info | jsonb, nullable | Source exchange/ticker entries; inspect shape before expanding |
| tickers, cusips | jsonb, nullable | Additional source identifiers; not guaranteed scalar strings |
| company_info | jsonb, nullable | Profile fields may include currency, location, website |
| company_links | jsonb, nullable | Source links; presence does not make a link a supplied citation |
| created_time, updated_time | timestamptz, nullable | Row maintenance times, not a dedicated market-price timestamp |

No current_price, high_52w, low_52w, pe_ratio, industry_macro or industry_basic columns are exposed here. Financial facts join to this master through the filing relations on company_id; see `references/datasets/financial-statements.md`.

## Example: resolved issuer profile

After resolving TSLA and confirming the columns, execute this SQL with the `run_query` payload in `references/datasets/sql-query-toolkit.md`:

```sql
SELECT company_id, company_name, stock_ticker, cik, is_delisted,
       industry_sector, industry, marketcap,
       company_info->>'currency' AS profile_currency, updated_time
FROM public.us_companies
WHERE stock_ticker = 'TSLA'
ORDER BY company_id
LIMIT 5
```

This example was executed read-only against the database. A symbol predicate is an example, not a universal identity resolver; use the confirmed company_id when symbols are ambiguous. No returned profile currency proves the currency of every associated numeric field.

## Example: same-industry candidates

This defines peers as current non-delisted issuers sharing the resolved company's stored industry. It does not imply comparable products, geography, size or profitability.

```sql
SELECT p.company_id, p.company_name, p.stock_ticker, p.industry_sector,
       p.industry, p.marketcap, p.marketcap_category
FROM public.us_companies AS p
JOIN public.us_companies AS target ON p.industry = target.industry
WHERE target.stock_ticker = 'TSLA'
  AND p.company_id <> target.company_id
  AND p.is_delisted = false
ORDER BY p.company_name, p.company_id
LIMIT 25
```

If the target industry is null, this returns no peers. Do not substitute a broad sector without saying the criterion changed. For an exact company_id, replace the target predicate with that resolved literal. Escape embedded single quotes by doubling them; do not execute unresolved placeholders.

## Market cap and coverage

Do not assume INR crore, USD millions, or a current valuation. The SQL schema gives no dedicated marketcap currency/as-of columns. The US company synchronization source leaves marketcap/category null for new rows and does not refresh those fields on ordinary updates. Consequently updated_time is not evidence that marketcap was refreshed. Confirm units and age with available source provenance before numerical thresholds, conversions, rankings or ratios; otherwise disclose the limitation. Missing marketcap does not mean zero or small-cap.

For a profile, show supported fields and omit or label unknowns. For a screen, state the universe, filters, missing-field exclusions and result cap. A bounded thematic shortlist is not an exhaustive screen. Do not infer inactive securities are currently tradable from historical filing coverage.
