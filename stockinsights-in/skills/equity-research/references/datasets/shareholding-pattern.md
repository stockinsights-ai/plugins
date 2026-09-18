# India Shareholding Data

## Purpose

Use this dataset to analyse disclosed Indian-company ownership snapshots, headline
ownership mix, and named holder rows. The data is point-in-time: compare the
same `filing_basis` across `as_of_date` values.

## Source and Tool

- Market: `in`
- SQL dialect: PostgreSQL
- Use read-only SQL with explicit columns, deterministic ordering, and a bounded
  result size. The toolkit does not support `$1`-style bind placeholders; escape
  SQL string literals instead.

Use `query_structured_financial_data`. Inspect the relevant tables in one call when a
required column is uncertain, then run the query:

```json
{ "body": { "action": "describe_tables", "table_names": ["in_shareholding_filings", "in_shareholding_categories"] } }
```

```json
{ "body": { "action": "run_query", "query": "SELECT ... LIMIT 50", "max_rows": 50 } }
```

## Tables and Relationships

| Table                        | Use                                                                      |
| ---------------------------- | ------------------------------------------------------------------------ |
| `in_companies`               | Resolve `stock_ticker` to `company_id`.                                  |
| `in_shareholding_filings`    | Select the authoritative shareholding snapshot and its observation date. |
| `in_shareholding_categories` | Read disclosed category totals and the category hierarchy.               |
| `in_shareholding_holders`    | Read individual/named holder rows for one filing.                        |

Join `in_companies.company_id` to `in_shareholding_filings.company_id`. Join
categories and holders to a filing with `shareholding_filing_id = filings.id`.
Use `company_id`, not the UUID primary key `in_companies.id`.

## Snapshot Rules

- Always filter `is_canonical = true`. It denotes the upstream-selected,
  authoritative version of a disclosure; it is not calculated by this API.
- `filing_basis` is stored as text. Supported values are `periodic`,
  `pre_listing`, and `capital_restructuring`. Use `periodic` for normal
  quarter-to-quarter or year-over-year ownership comparisons. Do not mix bases
  unless the question explicitly concerns the event-driven filing.
- `as_of_date` is the ownership observation date. It is distinct from filing and
  revision timestamps, which describe when the disclosure was published or
  revised.
- More than one canonical row can theoretically exist for a company, date, and
  basis. When selecting one snapshot, use the highest filing `id` as the tie
  breaker. Do not use `MAX` of percentages or shares to choose a filing.
- `reported_percentage` is a reported, rounded value. Preserve `NULL` as missing
  data, not zero; headline categories may not sum to exactly 100%.

## Headline Ownership Categories

Use these exact category codes from `in_shareholding_categories` for a standard
five-bucket ownership view:

| Category code                                 | Headline bucket          |
| --------------------------------------------- | ------------------------ |
| `shareholding_of_promoter_and_promoter_group` | Promoters                |
| `institutions_foreign`                        | FII                      |
| `institutions_domestic`                       | DII                      |
| `governments`                                 | Government               |
| `non_institutions`                            | Public/non-institutional |

These are taxonomy codes, not a database enum. Do not infer a headline bucket
from a category label or holder name. A new or unrecognised detailed code may
be valid under a newer taxonomy version.

## Holder-Row Rules

- Holder rows are disclosed source rows, not normalized beneficial ownership.
  Do not deduplicate identical names or assume renamed entities are the same
  holder without evidence.
- Exclude aggregate category rows with
  `LOWER(TRIM(holder_type)) = 'category'` when the question asks for named
  holders. Exclude masked names: empty string, `-`, `--`, and `******`.
- `holder_type = 'Promoter'` or `Promoter Group` is the strongest signal for
  promoter classification. Otherwise use `category_code`; for unfamiliar codes,
  follow `parent_category_code` in the categories table. If it still cannot be
  resolved, report the holder as unclassified rather than guessing.
- Intermediary/depositary rows may not represent a direct investor. Describe the
  reported row and category; do not automatically classify it as FII/DII from
  its name.

## Query Patterns

Resolve the company first, then reduce filings before joining categories or
holders. This avoids multiplying rows across the two child tables.

### Headline pattern over time

```sql
WITH filings AS (
  SELECT DISTINCT ON (f.as_of_date)
    f.id,
    f.as_of_date,
    f.total_shares,
    f.total_shareholders
  FROM in_shareholding_filings AS f
  JOIN in_companies AS c ON c.company_id = f.company_id
  WHERE c.stock_ticker = 'DRREDDY'
    AND f.is_canonical = true
    AND f.filing_basis = 'periodic'
    AND f.as_of_date >= DATE '2025-06-30'
  ORDER BY f.as_of_date, f.id DESC
)
SELECT
  f.as_of_date,
  f.total_shares,
  f.total_shareholders,
  MAX(c.reported_percentage) FILTER (
    WHERE c.category_code = 'shareholding_of_promoter_and_promoter_group'
  ) AS promoters_percentage,
  MAX(c.reported_percentage) FILTER (
    WHERE c.category_code = 'institutions_foreign'
  ) AS fii_percentage,
  MAX(c.reported_percentage) FILTER (
    WHERE c.category_code = 'institutions_domestic'
  ) AS dii_percentage,
  MAX(c.reported_percentage) FILTER (
    WHERE c.category_code = 'governments'
  ) AS government_percentage,
  MAX(c.reported_percentage) FILTER (
    WHERE c.category_code = 'non_institutions'
  ) AS public_percentage
FROM filings AS f
LEFT JOIN in_shareholding_categories AS c
  ON c.shareholding_filing_id = f.id
GROUP BY f.id, f.as_of_date, f.total_shares, f.total_shareholders
ORDER BY f.as_of_date;
```

### Largest named holders in one snapshot

```sql
WITH selected_filing AS (
  SELECT f.id, f.as_of_date
  FROM in_shareholding_filings AS f
  JOIN in_companies AS c ON c.company_id = f.company_id
  WHERE c.stock_ticker = 'DRREDDY'
    AND f.is_canonical = true
    AND f.filing_basis = 'periodic'
    AND f.as_of_date = DATE '2026-06-30'
  ORDER BY f.id DESC
  LIMIT 1
)
SELECT
  f.as_of_date,
  h.holder_name,
  h.holder_type,
  h.category_code,
  h.shares,
  h.reported_percentage
FROM selected_filing AS f
JOIN in_shareholding_holders AS h ON h.shareholding_filing_id = f.id
WHERE COALESCE(LOWER(TRIM(h.holder_type)), '') <> 'category'
  AND TRIM(h.holder_name) NOT IN ('', '-', '--', '******')
ORDER BY h.reported_percentage DESC NULLS LAST, h.shares DESC NULLS LAST, h.holder_name
LIMIT 25;
```

## Response Guidance

- State the `as_of_date` and `filing_basis` used.
- Report ownership changes in percentage points, not percentage growth.
- Distinguish changes in aggregate category ownership from changes in named
  holder rows. A stable aggregate can conceal transfers between disclosed legal
  vehicles.
- Say when a category or holder value is `NULL`, masked, or unclassified.
