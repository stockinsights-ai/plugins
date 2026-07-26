---
name: earnings-summary-workflow
description: Summarize one Indian company earnings-call transcript into a structured Markdown quick summary, sentiment analysis, business outlook, and potential risks. Use when the user provides a transcript or asks for a comprehensive summary of one identified earnings call. Ground every material point in the transcript.
---

# Earnings Summary

## Purpose

Analyze exactly one earnings-call transcript. Separate reported performance from forward-looking guidance and return a consistent, evidence-backed summary.

## Example Queries

- Summarize TCS FY26Q1 earnings call.
- Analyze the latest RELIANCE earnings transcript, including outlook and risks.
- What was management sentiment in the latest HDFCBANK call?

## Sources and Tools

- Market: `in`
- MCP server: `stockinsights-in`
- Primary tool: `get_filing_content`
- Supporting tool: `resolve_companies` for ambiguous company names
- Dataset reference: `../equity-research-workflow/references/datasets/filing-content.md`

Use only the supplied transcript or transcript returned by `get_filing_content`. Do not add facts from model memory or other filings.

## Retrieval Workflow

1. Identify the company and fiscal quarter. Use the latest available earnings transcript when the user does not specify a period.
2. If transcript text is supplied, analyze it directly. Otherwise read `../equity-research-workflow/references/datasets/filing-content.md`, resolve an ambiguous company identity, and call `get_filing_content` with the plain ticker, `filing_type: "earnings-transcript"`, and the requested `time_scope`.
3. Read all returned transcript pages, including prepared remarks and analyst Q&A. Track the page and `citation_link` for every material point.
4. Separate reported results from targets, expectations, plans, and aspirations.
5. Consolidate repeated statements while preserving important numbers, periods, segments, and management qualifications.
6. Return the structure under **Output Format**. Do not return JSON or wrap the answer in a code fence.

## Analysis Rules

### Quick Summary

- Group the most important takeaways under short, transcript-specific headers such as Financial Performance, Operational Highlights, Segment Performance, Strategic Developments, or Management Commentary.
- Include only supported categories and avoid a chronological recap.

### Sentiment

- List the most material positive and negative signals from prepared remarks and Q&A.
- Give a one-sentence overall assessment and a score from 1 to 5: `1` strongly negative, `2` negative, `3` mixed/neutral, `4` positive, `5` strongly positive.
- Base the score on the whole call. Do not infer sentiment from tone alone when facts or guidance indicate otherwise.

### Business Outlook

Use these exact headers: `Revenue Growth`, `Profitability`, `Market Expansion`, and `New Product Launches`.

Include only explicit forward-looking statements, targets, expected drivers, timelines, planned investments, and qualifications. Write `Not discussed.` when the transcript contains no relevant evidence.

### Potential Risks

- Classify a risk as `Major Risks` when management presents it as material, persistent, broad, or likely to meaningfully affect revenue, profitability, liquidity, execution, or strategy.
- Classify narrower, temporary, or manageable issues as `Minor Risks`.
- Do not invent risks from general industry knowledge. When materiality is unclear, use `Minor Risks` and avoid overstating certainty.

## Grounding and Response Guidance

- Every material point must be traceable to the transcript.
- Cite the closest page using `[Transcript, p. N](citation_link)`.
- Preserve uncertainty qualifiers and never convert guidance into achieved facts.
- Do not duplicate the same point within a category.
- If the transcript is unavailable or unusable, state that clearly without attempting the analysis.

## Output Format

```markdown
# <Company> Earnings Summary — <Fiscal Quarter>

## Quick Summary

### <Relevant Header>

- <Point>

## Sentiment

### Positive

- <Point>

### Negative

- <Point>

**Overall sentiment:** <Assessment>

**Overall sentiment score:** <1-5>/5

## Business Outlook

### Revenue Growth

- <Point or "Not discussed.">

### Profitability

- <Point or "Not discussed.">

### Market Expansion

- <Point or "Not discussed.">

### New Product Launches

- <Point or "Not discussed.">

## Potential Risks

### Major Risks

- <Point or "Not discussed.">

### Minor Risks

- <Point or "Not discussed.">
```

Use transcript-specific subheaders only under `Quick Summary`. Keep all other headers exactly as shown and omit placeholder text.
