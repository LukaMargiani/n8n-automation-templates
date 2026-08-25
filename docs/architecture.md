# Patterns across the five workflows

These were built over one engagement against one Airtable base, so they share a house style. The recurring decisions are more transferable than any individual workflow.

## Airtable is the interface, not just the database

Every workflow reads its inputs from a table and writes its outputs to a table. That is not a storage choice, it is an interface choice. A non-technical team can:

- change what the Job Radar monitors, by adding a row to Organisations
- trigger a research run, by typing a prompt into Research Requests
- approve content, by changing a Status field
- see whether anything is broken, by opening Automation Errors

None of that requires n8n access, and none of it can break a workflow. The alternative — configuration living inside nodes — means every small change is a request to the person who built it, and that person eventually leaves.

## The model judges, deterministic logic decides

In every workflow where an LLM appears, it produces a *value* and a plain n8n node acts on it:

| Workflow | Model produces | Node decides |
|---|---|---|
| EU Funding Radar | `score` 0-10 | IF: `score >= 7` |
| Contact Enricher | extracted people | IF on Tavily confidence, upstream |
| Spotlight | `status: no_news` | IF: skip and re-loop |
| Job Radar | structured roles | merge against known vacancies |

Thresholds sit in nodes where they are visible and tunable. Deduplication is a merge, not a prompt instruction. When behaviour needs to change, you change a number, not a paragraph.

## Structured output everywhere

Every LLM node uses `response_format: json_object` and a prompt that specifies the exact schema. No free-text parsing, no regex over prose. When a model returns something malformed, it fails at the parse rather than writing garbage into the CRM.

## Failure is a state, not an absence

Three of the five write an explicit failure record: `Failed - Low Confidence` for a research request nothing useful came back for, and an Automation Errors row for anything that throws. Silence is the worst possible output, because it is indistinguishable from "nothing to report".

## Idempotency by design

Every workflow can be re-run safely:

- Job Radar merges against existing vacancies before inserting
- Contact Enricher upserts on name plus organisation, and flips requests to `Completed`
- Spotlight only stamps `Last Featured Date` when a draft was actually stored
- EU Funding Radar filters on a 7-day ingest window matching its weekly schedule

This is worth designing on day one. Retrofitting it after the first duplicate-notification incident is much harder.

## Loop-with-continue rather than filter-then-process

All four scheduled workflows use `splitInBatches` with the IF branches feeding *back* into the loop node. Items that fail a check return to the loop instead of falling out of the pipeline. It keeps a single execution per run, makes the n8n execution view readable, and means one bad item cannot halt a batch.

## Cost is an architectural input

The Spotlight Generator uses Jina AI because it runs monthly and fits a free tier. The Job Radar uses Firecrawl because it needs crawl-and-discover. The EU Funding Radar filters on deadline and freshness *before* scoring, so tokens are only spent on calls that can still be acted on. Choosing the cheaper tool where it suffices is a design decision, not a compromise.
