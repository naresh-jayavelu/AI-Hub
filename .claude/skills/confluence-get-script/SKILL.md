---
name: confluence-get-script
description: 'Use when fetching Confluence page content via the REST API PowerShell script. Retrieves page title, body, labels, child pages, and attachments. Converts Confluence storage format to clean markdown. Saves formatted markdown and images to the task-specific context folder. Does NOT require MCP server configuration. Cannot search by keywords - requires specific page URLs or IDs. Triggers on: Confluence page URLs, page IDs, requests to "fetch the docs", "get Confluence page", or "look up the spec".'
---

# Confluence Get Script

Fetch Confluence page content using the REST API PowerShell script and save
structured context to the task folder.

## Prerequisites

The API token file must exist at `.github/personal-atlassian-api-token.txt`
with format: `your-email@your-company.com:your_api_token`

## Procedure

### Step 1: Parse Input

Extract Confluence page URL(s) or ID(s) from the argument. Accepted formats:

- Page ID: `1481845651`
- Full URL: `https://your-company.atlassian.net/wiki/spaces/SPACE/pages/1481845651`
- Multiple pages: space-separated URLs or IDs

If no pages are provided, ask the developer for specific page URLs or IDs.
This script cannot search Confluence by keywords.

### Step 2: Run the Script

```powershell
.\.github\skills\confluence-get-script\scripts\confluence-get.ps1 -PageUrls "{URL_OR_ID_1}" "{URL_OR_ID_2}" -TaskId "{TASK-ID}"
```

This will:

- Fetch each page's content (title, body, labels, child pages, attachments)
- Convert Confluence storage format to clean markdown
- Strip diagram macros and replace with `[DIAGRAM: ...]` markers
- Convert tables to structured text format
- Decode HTML entities
- Save each page to: `.github/context/task-specific/{TASK-ID}/confluence-{pageId}.md`
- Download all attached images to the folder

### Step 3: Read and Return Context

After the script completes, read the saved markdown files from
`.github/context/task-specific/{TASK-ID}/` and return the content.

Extract key information:
- Architectural decisions
- API contracts
- Data models
- Non-functional requirements
- Relevant diagrams (noted as placeholders; actual images saved locally)

## Error Handling

| Error                | Action                                                             |
| -------------------- | ------------------------------------------------------------------ |
| Token file missing   | Ask developer to create `.github/personal-atlassian-api-token.txt` |
| Invalid token format | Token must be `email:api_token` format                             |
| 401 Unauthorized     | Token is invalid or expired; ask developer to update it            |
| 404 Not Found        | Page ID/URL does not exist or no access; ask developer to verify   |
| No pages provided    | Ask developer for specific Confluence page URLs or IDs             |

## DO

- Accept multiple page URLs/IDs in a single invocation
- Use the ticket ID as TaskId when fetching pages related to a ticket
- Note child pages listed in the output for potential follow-up fetching

## DO NOT

- Modify any Confluence page
- Expose the API token in output
- Attempt keyword search (this script fetches specific pages only)
- Fetch more than 5 pages in a single invocation to avoid excessive API calls
