---
name: plugin-skill-creator
description: Create plugin-related data retrieval skills. Use when Codex needs to draft or update a plugin skill that retrieves data from MCP tools or similar plugin data sources, including StockInsights-style skills for filings, company data, announcements, calendars, or search-backed research workflows.
---

# Plugin Skill Creator

Use this skill to create or revise a plugin-scoped data retrieval `SKILL.md`.

v1 supports only data retrieval skills. Do not scaffold a full plugin, plugin manifest, MCP server config, assets, marketplace entry, or setup/troubleshooting skill unless the user explicitly asks for a different skill type.

## Inputs to Gather

Gather these from the developer prompt, existing tool docs, MCP schemas, or verified local files:

- Skill name and trigger intent.
- Market: India (`in`) or US (`us`).
- Example user queries.
- Data being retrieved and why it is useful.
- Important data variations at a high level.
- Source boundary, such as the plugin/MCP server that must be used.
- Primary tools and any required resolution/search/filter sequence.
- Input shape, required fields, useful defaults, and a few misuse-preventing examples.
- Expected response shape and how to handle response variations.
- Developer-provided nuances, constraints, and LLM instructions.

Do not invent nuanced behavior. Use only developer-provided instructions or verified source/tool behavior.

## Market and Location

Create the generated skill in the plugin folder for its market:

- India / `in`: `stockinsights-in/skills/<skill-name>/SKILL.md`
- US / `us`: `stockinsights-us/skills/<skill-name>/SKILL.md`

If the market is not clear from the user request, ask for it before creating files. Do not place market-specific retrieval skills in `.agents/skills`; reserve `.agents/skills` for creator or repo-level agent skills.

Use market-specific tool names, payload fields, and source behavior from the relevant plugin or MCP schema. Do not copy India-specific assumptions into US skills or US-specific assumptions into India skills.

## Frontmatter Rules

Generate valid skill frontmatter with only:

```yaml
---
name: skill-name
description: Clear trigger description with the task and when to use it.
---
```

Rules:

- Use lowercase hyphen-case for `name`.
- Put all trigger-critical wording in `description`.
- Do not put example queries in frontmatter.
- Do not add custom YAML keys such as `examples` unless the active skill validator explicitly allows them.

## Body Structure

Use this structure by default. Keep it concise and remove sections that do not apply.

```markdown
# Human Readable Skill Title

## Purpose

Describe the data the skill retrieves, the user jobs it supports, and the main answer types it should produce.

## Example Queries

- Example question 1.
- Example question 2.
- Example question 3.

## Data

Explain:

- what the data represents,
- why it is useful,
- important variations the agent may see.

Keep variations high level. Do not document every schema branch unless that detail prevents likely misuse.

## Sources and Tools

Name the source boundary and the primary tools.

- Market: `in` or `us`
- Skill location: `stockinsights-in/skills/skill-name` or `stockinsights-us/skills/skill-name`
- MCP server or plugin: `server-name`
- Primary tools: `tool_one`, `tool_two`
- Required first step, if any: company/entity resolution, date selection, ticker normalization, etc.

If the requested source or tool is unavailable, report the blocker plainly instead of silently switching sources.

## Input Guidance

Describe the input contract as a guide, not an exhaustive reference.

Include:

- required shape or required fields,
- important optional fields,
- defaults that affect behavior,
- 1-3 compact payload examples when useful.

Avoid copying large schemas into the skill body. Move large schema notes to a reference file only if the user asks for a broader skill package.

## Retrieval Workflow

Give the minimum sequence needed to use the tools correctly.

Cover:

- how to resolve user-provided identifiers,
- how to choose the relevant tool or filter,
- how to handle pagination, date scope, or result limits when applicable,
- when to run additional calls to gather enough evidence.

Keep workflow rules source-specific. Do not add generic retrieval heuristics that the developer did not ask for.

## Response Guidance

Describe what the agent should expect from tool responses and how to use them.

Include:

- fields or metadata that identify the source,
- citation or evidence fields when available,
- response variations the agent should expect,
- how to answer when evidence is partial, conflicting, or unavailable.

Do not enumerate every possible response contract branch. Explain the meaningful variations and the handling rule.

## Nuances

Include only developer-provided or verified nuances.

Examples of valid nuance categories:

- source must not be substituted,
- specific tool exposure failures,
- known defaults,
- important calculation or citation rules,
- known schema pitfalls.

## Validation Checklist

- Frontmatter has only `name` and `description`.
- Market is explicit, and the skill is created under the matching `stockinsights-in/skills` or `stockinsights-us/skills` folder.
- Example queries are in the body, not frontmatter.
- The data source boundary and primary tools are named.
- Input guidance is practical but not exhaustive.
- Response guidance explains expected variations without over-documenting schemas.
- Nuances come from the developer or verified source behavior.
- Missing tools or unavailable data are handled explicitly.
```

## Quality Bar

Prefer small, direct skills that another agent can use immediately. The generated retrieval skill should make the correct data path obvious without becoming an API manual.

Keep APIs, tool names, and defaults explicit. Remove generic advice that does not change how the retrieval should be performed.
