# n8n Automation Templates

Six production n8n workflows, generalised into templates you can import and run on your own instance.

Five were built for a European policy non-profit over a two-month engagement, to replace administrative work that was eating hours every week: manually checking member websites for job postings, watching EU funding portals for relevant grants, researching contact details by hand, and drafting the same kind of LinkedIn post every month. All five ran unattended against one Airtable base, and the fifth exists so the team could tell when the other four broke.

The sixth, Job Scout, is my own. It runs every morning and does the reading half of a job search: it checks each new listing against a CV and keeps track of the applications already sent.

Every client identifier, personal detail, API key, base id and pinned execution record has been removed. What is left is the architecture.

---

## The workflows

| Workflow | What it does | Interesting because |
|---|---|---|
| [EU Funding Radar](docs/eu-funding-radar.md) | Polls the EU funding portal, has an LLM score each call 0–10 for mission fit, pushes the good ones to Teams as Adaptive Cards | The relevance cutoff lives in an IF node, not in the prompt |
| [Self-Healing Job Radar](docs/self-healing-job-radar.md) | Watches career pages for new vacancies and extracts them as structured JSON | Repairs its own source data when a career-page URL dies |
| [GDPR Contact Enricher](docs/gdpr-contact-enricher.md) | Turns a plain-language research request into verified, compliant contact records | Confidence gate before enrichment; async callback handled with a Wait node |
| [Spotlight Content Generator](docs/spotlight-content-generator.md) | Drafts a monthly LinkedIn post about the organisation you have featured least recently | Human approves every draft before anything publishes |
| [LLM Error Logger](docs/llm-error-logger.md) | Catches any failed execution anywhere in the instance and logs it in readable form | Two nodes. The reason the other four survived handover |
| [Job Scout](docs/job-scout.md) | Collects listings from ten job sources daily, has an LLM check each description against your CV, emails the jobs you qualify for and logs your applications from Gmail | The model scores; deterministic backstops catch the experience and seniority calls it gets wrong |

Read [docs/architecture.md](docs/architecture.md) for the patterns these share, and [docs/airtable-schema.md](docs/airtable-schema.md) for the tables they expect. Job Scout uses an n8n Data Table instead of Airtable; its columns are listed in [docs/job-scout.md](docs/job-scout.md).

---

## Setup

**1. Import.** In n8n: *Workflows → Import from File*, one per file in `workflows/`.

**2. Build the Airtable base.** [docs/airtable-schema.md](docs/airtable-schema.md) lists every table and field. Start with `Organisations` and `Automation Errors`; the rest are per-workflow. For Job Scout, create an n8n Data Table named `Job Scout Jobs` instead (columns in its doc).

**3. Reconnect credentials.** Every credential in these files reads `REPLACE_WITH_YOUR_CREDENTIAL_ID`. Open each red-flagged node and pick your own credential from the dropdown. You will need:

| Service | Used by | Notes |
|---|---|---|
| Airtable (Personal Access Token) | the five client workflows | scoped to your base |
| OpenAI | four | any chat model; these were built on `gpt-5-mini` |
| Dropcontact | Contact Enricher | paid, GDPR-compliant enrichment |
| Anthropic | Job Scout | built on Claude Haiku 4.5 |
| Gmail (OAuth2) | Job Scout | only reads mail: finds application confirmations, rejections and invites |
| SMTP | Job Scout | sends the daily digest |
| Adzuna, RapidAPI (JSearch), Firecrawl | Job Scout | optional; generic HTTP credentials, formats in [docs/job-scout.md](docs/job-scout.md) |

**4. Set environment variables.** API keys that are sent as raw headers read from `$env`, so set these on your n8n instance (`.env` for Docker, or the host environment):

```bash
FIRECRAWL_API_KEY=      # Self-Healing Job Radar
TAVILY_API_KEY=         # GDPR Contact Enricher
DROPCONTACT_API_KEY=    # GDPR Contact Enricher (webhook subscription node)
```

Job Scout needs none of these: its HTTP keys live in n8n credentials, and Jooble's key, which has to go inside the URL, is a field in its `Your Settings` node.

**5. Fill the placeholders.** Search each workflow for `YOUR_` and replace:

| Placeholder | Where | What to put |
|---|---|---|
| `YOUR_AIRTABLE_BASE_ID` | the five client workflows | your base id (`app…`) |
| `YOUR_*_TABLE_ID` | per workflow | the matching table id (`tbl…`) |
| `YOUR_TEAMS_WEBHOOK_URL` | EU Funding Radar | your incoming-webhook URL — treat it as a secret, it carries its own signature |
| `YOUR_ERROR_WORKFLOW_ID` | EU Funding Radar, Spotlight, Job Scout | the id of your imported LLM Error Logger |
| `{{ORGANISATION_NAME}}`, `{{ORGANISATION_MISSION}}` | EU Funding Radar, Spotlight | your organisation and a one-line mission — the LLM scores against this |
| `PASTE YOUR CV HERE`, `you@example.com`, `YOUR_JOOBLE_API_KEY` | Job Scout, `Your Settings` node | your CV as plain text, your digest addresses, and a Jooble key or leave it to skip Jooble |

**6. Wire up error handling.** In each workflow: *Settings → Error Workflow → LLM Error Logger*. Do this before you enable any schedule.

**7. Run once manually** before switching on a trigger. Every one of these writes to Airtable or a Data Table.

---

## A note on what these are

These are templates derived from systems I built, not the client's deployment. The prompts, control flow and architecture are mine and are reproduced faithfully. Everything that belonged to the client is gone: their base, their table ids, their contact taxonomy, their member list, and the pinned execution data that carried real people's names and email addresses.

Job Scout was never client work; it is the template version of the one I run for my own job search. My CV, addresses, keys and saved jobs are not in it, and the CV field ships empty.

If you are adapting the Contact Enricher, note that it processes personal data. Dropcontact is used specifically because it enriches against a GDPR-compliant source rather than scraping addresses, and the workflow tags every record it creates so you can always tell automated entries from human ones. Keep both of those properties if you fork it.

## Licence

MIT. See [LICENSE](LICENSE).
