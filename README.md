# Automated Weekly Multi-Report with n8n

Automated workflow that runs 4 parallel SQL queries against PostgreSQL, generates multiple Excel reports, builds a dynamic HTML summary email, and sends everything via Microsoft Outlook every Monday morning.

## Workflow Architecture

```
Schedule Trigger (Monday 3:10 AM)
    │
    ├──► Query - Scheduled Visits ──► Transform ──► File (.xlsx) ──────────────┐
    │                                                                            │
    ├──► Query - Visits with Report ──► Transform ──► File (.xlsx) ─────────────┤──► Merge Files
    │                                           └──► Count ──────────────────┐  │
    │                                                                         │  │
    ├──► Query - Requirements ──────► Transform ──► File (.xlsx) ─────────────┤──┘
    │                                          └──► Count ──────────────────┤
    │                                                                        │
    └──► Query - Follow-up Visits ──► Transform ──► File (.xlsx) ────────────┤
                                                └──► Count ──────────────────┘
                                                                             │
                                                                    Merge Counts
                                                                             │
                                                                    Build HTML Email
                                                                             │
                                                              Merge Email + Files
                                                                             │
                                                                    Send Email (Outlook)
```

## What it does

1. **Triggers automatically** every Monday at 3:10 AM
2. **Runs 4 parallel queries** simultaneously against PostgreSQL, each focused on a different activity type from the last 7 days
3. **Transforms each result** with JavaScript, formatting dates and mapping to readable column names
4. **Generates 3 Excel files** as attachments
5. **Counts records** from each query to build a summary
6. **Builds a dynamic HTML email body** with a summary table showing counts per category
7. **Merges all binary files and JSON** into a single item
8. **Sends the email** via Microsoft Outlook with all Excel files attached

## Report Types

| Report | Description |
|---|---|
| Visits with Report | Evaluation visits that had a report submitted in the last 7 days |
| Requirements | Records with a new requirement registered in the last 7 days |
| Follow-up Visits | Follow-up visits with report submitted in the last 7 days |

## Key Technical Patterns

**Parallel execution**: The Schedule Trigger fans out to 4 independent query branches that run simultaneously.

**Binary file merging**: A dedicated JavaScript node merges all binary Excel files into a single item so they can all be attached to one email.

**Dynamic HTML email**: Counts from each branch are merged and used to build an HTML summary table injected into the email body.

**DISTINCT ON**: Each SQL query uses PostgreSQL's `DISTINCT ON` to get the latest record per entity, avoiding duplicates.

## Workflow Nodes

### Schedule Trigger
- **Frequency**: Weekly, Monday at 3:10 AM
- **Fan-out**: Triggers all 4 query branches simultaneously

### Query Nodes (PostgreSQL)
- **Query - Scheduled Visits**: Visits scheduled in the last 7 days without a report yet
- **Query - Visits with Report**: Evaluation visits that received a report in the last 7 days
- **Query - Requirements**: Records with a new requirement registered in the last 7 days
- **Query - Follow-up Visits**: Follow-up visits with a report submitted in the last 7 days

### Transform Nodes (JavaScript)
Each transform node formats dates to `YYYY-MM-DD` and maps raw DB column names to readable labels.

### Count Nodes (JavaScript)
Each count node returns the total number of records from its branch for use in the summary email.

### Merge Nodes
- **Merge - Files**: Collects all Excel files from the 4 branches
- **Merge Binary Files**: Combines all binary files into a single item
- **Merge - Counts**: Collects all count totals
- **Merge - Email + Files**: Combines the HTML email body with the binary files
- **Merge JSON + Binary**: Final merge that produces a single item with both JSON and binary data

### Build HTML Email
Generates a dynamic HTML table with the weekly summary counts, ready to inject into the email body.

### Send Email (Microsoft Outlook)
Sends the HTML email with all 3 Excel files attached.

## Prerequisites

- **n8n** instance running
- **PostgreSQL** database
- **Microsoft Outlook** account with OAuth2 configured in n8n

## Required Credentials in n8n

| Credential | Node | Description |
|---|---|---|
| `postgres` | All Query nodes | PostgreSQL connection |
| `microsoftOutlookOAuth2Api` | Send Email | Microsoft Outlook OAuth2 |

## Variables to Configure

| Variable | Location | Description |
|---|---|---|
| `YOUR_POSTGRES_CREDENTIAL_ID` | Query nodes | Your PostgreSQL credential ID |
| `YOUR_OUTLOOK_CREDENTIAL_ID` | Send Email node | Your Outlook credential ID |
| `recipient1@yourdomain.com` | Send Email node | Report recipients |
| Table and schema names | SQL queries | Adapt to your own database schema |
| `visit_type` values | SQL WHERE clauses | Adapt to your own visit type values |
| `record_type` values | SQL WHERE clauses | Adapt to your own record type values |

## Adapting the SQL Queries

Replace generic names with your actual schema:

| Generic name | Replace with |
|---|---|
| `your_schema.records` | Your main records table |
| `your_schema.visits` | Your visits table |
| `your_schema.documents` | Your documents table |
| `your_schema.requirements` | Your requirements table |
| `get_officer_name()` | Your function to resolve officer names |
| `Type_A`, `Type_B`, `Type_C` | Your actual record type values |
| `visit_json`, `doc_json`, `req_json` | Your actual JSONB column names |

## Potential Improvements

- Add error handling to notify on query failure
- Add a filter node to skip sending if all counts are zero
- Split reports by department or record type
- Add charts or graphs to the HTML email
- Store weekly counts in a table for historical tracking
