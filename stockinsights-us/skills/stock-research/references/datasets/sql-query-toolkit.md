# SQL Query Toolkit

## Purpose

Shared SQL retrieval sequence for US company and statement questions, through `query_structured_financial_data` on the US MCP server. The tool interface matches India; relations, columns and financial semantics do not.

## Workflow

1. Inspect every relevant relation in one call, using the exact names the tool's `table_names` schema lists. For the US company master:

```json
{"body":{"action":"describe_tables","table_names":["us_companies"]}}
```

2. Build one read-only PostgreSQL SELECT using confirmed columns, joins and units. Call `query_structured_financial_data` with `body.action` set to `run_query` and `body.query` holding the statement; set `body.max_rows` explicitly. The field is `query`, not `sql`; the default row cap is 100.
3. Inspect returned row counts, truncation and errors before interpreting the result. Refine only when needed to answer the question or correct a rejected input. After a second input rejection, report the failure.

`describe_tables` and `run_query` are values of `body.action`, not separate tools. Never execute DML or DDL, use SELECT *, invent schema names, or send unbound SQL placeholders. Select relevant columns, escape literals, add deterministic ordering and LIMIT. Do not repeatedly inspect a table or run speculative test queries. Keep all relations within the selected US catalog.

## Interpretation

Schema discovery describes accessible relations, not complete data coverage. Table existence does not prove that a given company or period has rows. Read `references/datasets/financial-statements.md` before interpreting reported facts. An empty result is not a zero value; a transport or permission error is not an empty dataset.

Do not invent citation markers for SQL rows. Preserve returned provenance and state source period, scope and units. SQL results carry no citation link; cite them as described in `references/datasets/financial-statements.md`.

## Execution envelope and bounds

After table inspection, a complete tool payload is:

```json
{"body":{"action":"run_query","query":"SELECT company_id, company_name, stock_ticker FROM public.us_companies WHERE stock_ticker = 'TSLA' ORDER BY company_id LIMIT 5","max_rows":5}}
```

Use an explicit integer max_rows from 1 to 1000, normally 50 or less. Omitted max_rows defaults to 100. The backend still caps output at 1000 and uses a 15-second statement timeout. These caps bound returned rows, not the amount of database work: keep predicates selective and avoid broad fact-table scans.

The result includes columns, rows, row_count, truncated and execution_time_ms. Numbers represented as strings can be exact PostgreSQL numeric/bigint values; do not treat them as missing or round them prematurely. Alias expressions and duplicate column names explicitly. An ambiguous-column error is a query-shape failure, not missing coverage.

SQL LIMIT and the API row cap interact: a query containing LIMIT 25 can return 25 rows and truncated=false even when more eligible rows exist. The flag says the API did not discard another fetched row; it does not establish an exhaustive universe. For a question requiring an exact count, use an explicitly scoped COUNT query rather than counting a limited sample.

## Recovery and evidence

- Invalid input: correct the payload once from the schema. Do not call run_query or describe_tables as a standalone tool.
- Query error: use the returned detail and already-inspected metadata; do not guess alternate schemas.
- Timeout: narrow the issuer/filing/period predicate and selected output; adding LIMIT alone may not reduce expensive joins or aggregation.
- Permission/configuration/transport failure: report unavailable access. Do not reinterpret the failure as no companies or no facts.
- Empty rows: report no matching data under the actual filters, with the documented filing-search fallback where one applies.

The catalog is curated discovery, not a grant of access to other relations or markets. Do not explore unrelated schemas, session settings, roles or system catalogs. Use only the US relations described in these references.
