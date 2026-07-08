---
name: jira-get-script
description: 'Use when fetching Jira ticket context via the REST API PowerShell script. Retrieves ticket summary, description, acceptance criteria, linked issues, sub-tasks, and attached images. Saves formatted markdown and images to the task-specific context folder. Does NOT require MCP server configuration. Triggers on: Jira ticket IDs (PROJECT-123), Jira URLs, requests to "fetch ticket", "get Jira context", or "look up issue".'
---

# Jira Get Script

Fetch Jira ticket details using the REST API PowerShell script and save
structured context to the task folder.

## Prerequisites

The API token file must exist at `.github/personal-atlassian-api-token.txt`
with format: `your-email@your-company.com:your_api_token`

## Procedure

### Step 1: Parse Input

Extract the Jira issue key from the argument. Accepted formats:
- Direct ID: `PROJECT-123`
- Full URL: `https://your-company.atlassian.net/browse/PROJECT-123`

If a TaskId is not provided separately, use the issue key as the TaskId.

### Step 2: Run the Script

```powershell
.\.github\skills\jira-get-script\scripts\jira-get.ps1 -IssueKey "{ISSUE-KEY}" -TaskId "{TASK-ID}"
```

This will:
- Fetch the ticket via Jira REST API (summary, description, acceptance criteria, linked issues, sub-tasks, attachments)
- Create folder: `.github/context/task-specific/{TASK-ID}/`
- Save formatted markdown to: `jira-{ISSUE-KEY}.md`
- Download all attached images to the folder

### Step 3: Read and Return Context

After the script completes, read `.github/context/task-specific/{TASK-ID}/jira-{ISSUE-KEY}.md`
and return the content. Also note any documentation URLs found in the description
or linked issues for downstream use.

## Error Handling

| Error | Action |
|-------|--------|
| Token file missing | Ask developer to create `.github/personal-atlassian-api-token.txt` |
| Invalid token format | Token must be `email:api_token` format |
| 401 Unauthorized | Token is invalid or expired; ask developer to update it |
| 404 Not Found | Issue key does not exist; ask developer to verify |

## DO

- Normalize issue keys to uppercase `PROJECT-NUMBER` format
- Use the issue key as TaskId if no separate TaskId is provided
- Scan the output for documentation URLs that may be useful for downstream context

## DO NOT

- Modify any Jira ticket
- Expose the API token in output
- Run the script without a valid issue key
