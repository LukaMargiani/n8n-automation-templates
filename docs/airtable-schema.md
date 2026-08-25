# Airtable schema

One base, six tables. Build the two shared tables first; the rest are per-workflow.

## `Organisations` — shared by Job Radar and Spotlight

| Field | Type | Notes |
|---|---|---|
| `Organisation Name` | Single line text | |
| `Website URL` | URL | The only field you must seed |
| `Career Page` | URL | Discovered and written back by the Job Radar |
| `Last Featured Date` | Date | Written by Spotlight; drives fair rotation |
| `Content Drafts` | Link to Content Drafts | |
| `Vacancies` | Link to Vacancies | |

## `Automation Errors` — used by every workflow

| Field | Type | Notes |
|---|---|---|
| `Workflow Name` | Single line text | |
| `Error Message` | Long text | |
| `Related Record ID` | URL | The n8n execution URL — click straight to the failure |
| `Timestamp` | Date | |
| `Severity Level` | Single select | Info / Warning / Error / Critical. Set by hand |
| `Status` | Single select | Open / Investigating / Resolved |

A grid view filtered to `Status = Open` is the system-health dashboard.

## `Vacancies` — Job Radar

| Field | Type | Notes |
|---|---|---|
| `Job Title` | Single line text | Half of the dedup key |
| `Summary/Snippet` | Long text | Other half of the dedup key |
| `Link` | URL | |
| `Deadline` | Single line text | Text, not Date — pages write "rolling" as often as a date |
| `Organisation` | Link to Organisations | |
| `Status` | Single select | New / Approved / Posted / Rejected |
| `Date Found` | Date | |

## `Content Drafts` — Spotlight

| Field | Type | Notes |
|---|---|---|
| `Topic` | Single line text | |
| `Summary` | Long text | |
| `Draft Text` | Long text | The LinkedIn post |
| `Link` | URL | Required — the IF node rejects drafts without one |
| `Status` | Single select | To Review / Approved |
| `Related Organisation` | Link to Organisations | |

## `Research Requests` — Contact Enricher

| Field | Type | Notes |
|---|---|---|
| `Prompt` | Single line text | What staff type, e.g. "policy experts in Poland" |
| `Target Country` | Single select | Optional |
| `Status` | Single select | Pending / Completed / Failed - Low Confidence |
| `Contacts` | Link to Contact List | |
| `Time Created` | Created time | |

The workflow polls for `Status = Pending`, so creating a row *is* the trigger.

## `Contact List` — Contact Enricher

| Field | Type | Notes |
|---|---|---|
| `Name` | Single line text | Upsert key, with Organisation |
| `Organization/Institution/Department/Region/Sector` | Single line text | Upsert key |
| `Email/Link` | Single line text | From Dropcontact only, never inferred |
| `Country` | Single select | |
| `Role/Category` | Single select | Automated records are tagged `New Contact n8n` |
| `Interests/Tags` | Single line text | Comma-joined |
| `Phone Number` | Single line text | |
| `Notes/Address/Comments` | Single line text | |
| `Research Requests` | Link to Research Requests | Provenance: which prompt produced this contact |
| `Time Created` | Created time | |
