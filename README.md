# Automated Weekly Report with n8n

Automated workflow that queries pending records from a PostgreSQL database, generates an Excel report, and sends it by email via Microsoft Outlook every Monday morning.

## Workflow Architecture

```
Schedule Trigger (Monday 4:10 AM)
    │
    ▼
Execute SQL Query (PostgreSQL)
    │
    ▼
Code in JavaScript (data transformation)
    │
    ▼
Convert to File (XLSX)
    │
    ▼
Send Email (Microsoft Outlook)
```

## What it does

1. **Triggers automatically** every Monday at 4:10 AM
2. **Queries PostgreSQL** to retrieve all pending records, including those with and without modifications, using CTEs for clean and modular SQL
3. **Transforms the data** with JavaScript, formatting dates and mapping fields to readable column names
4. **Generates an Excel file** (.xlsx) with the results
5. **Sends the report by email** via Microsoft Outlook with the file attached

## Workflow Nodes

### 1. Schedule Trigger
- **Frequency**: Weekly
- **Day**: Monday
- **Time**: 4:10 AM
- **Function**: Automatically starts the workflow every Monday morning so the report is ready at the start of the work week.

### 2. Execute SQL Query (PostgreSQL)
- **Type**: `n8n-nodes-base.postgres`
- **Operation**: Execute Query
- **Function**: Queries pending records using CTEs to separate logic into reusable blocks. Handles two scenarios:
  - Records **without modifications**
  - Records **with modifications** (uses the latest modification state)

**SQL structure:**
```sql
WITH pending_states AS (...),
     allowed_types AS (...),
     start_records AS (...),
     approval_records AS (...),
     latest_modification AS (...)

SELECT ... FROM main_table          -- without modification
UNION ALL
SELECT ... FROM main_table          -- with modification
JOIN latest_modification ...
```

### 3. Code in JavaScript
- **Type**: `n8n-nodes-base.code`
- **Function**: Transforms raw database results into clean, readable column names and formats all dates to `YYYY-MM-DD`.

### 4. Convert to File
- **Type**: `n8n-nodes-base.convertToFile`
- **Format**: XLSX
- **Function**: Converts the transformed data into an Excel file ready to attach.

### 5. Send Email (Microsoft Outlook)
- **Type**: `n8n-nodes-base.microsoftOutlook`
- **Function**: Sends the Excel report to the configured recipients with an HTML email body.

## Prerequisites

- **n8n** instance running
- **PostgreSQL** database with your data tables
- **Microsoft Outlook** account with OAuth2 configured in n8n

## Required Credentials in n8n

| Credential | Node | Description |
|---|---|---|
| `postgres` | Execute SQL Query | PostgreSQL connection |
| `microsoftOutlookOAuth2Api` | Send a message | Microsoft Outlook OAuth2 |

## Variables to Configure

| Variable | Location | Description |
|---|---|---|
| `YOUR_POSTGRES_CREDENTIAL_ID` | SQL node | Your PostgreSQL credential ID |
| `YOUR_OUTLOOK_CREDENTIAL_ID` | Email node | Your Outlook credential ID |
| `recipient1@yourdomain.com` | Email node | Report recipients |
| Table and column names | SQL query | Adapt to your own schema |
| State and type values | SQL CTEs | Adapt to your own data values |

## Adapting the SQL Query

The SQL query uses generic names. Replace them with your actual schema:

| Generic name | Replace with |
|---|---|
| `your_schema.your_main_table` | Your main records table |
| `your_schema.your_docs_table` | Your documents/autos table |
| `your_schema.your_resolutions_table` | Your resolutions table |
| `your_schema.your_modifications_table` | Your modifications table |
| `get_applicant_name()` | Your function to get applicant name |
| `get_location_name()` | Your function to get location name |
| `State_1`, `State_2`... | Your actual pending state values |
| `Type_A`, `Type_B`... | Your actual record type values |

## Tags

```
n8n
postgresql
automation
reporting
excel
microsoft-outlook
scheduled-workflow
data
javascript
```

## Potential Improvements

- Add error handling node to notify if the query fails
- Add a filter node to exclude records older than N days
- Send different reports to different teams based on record type
- Add a summary count at the top of the email body
