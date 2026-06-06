---
description: >
  Pull and analyze a ticket from Azure DevOps or Jira. Extracts the full context including
  title, description, acceptance criteria, child tasks, linked PRs, related items, and
  sprint/iteration information. Use whenever a user shares a ticket ID (e.g., "ADO:12345",
  "#12345", "NEXUS-456") or URL and asks about its content, status, linked items, or
  acceptance criteria. Also trigger for phrases like "analyze this ticket", "what does this
  story require", "show me the acceptance criteria for", "what's linked to this ticket", or
  "summarize this work item". Supports both ADO and Jira; detect the source from the ID
  format or URL.
---

# dm-analyze

Fetch and analyze a ticket from Azure DevOps or Jira, producing a structured analysis snapshot.

## Defaults

- **ADO organisation**: configured in `.mcp.json` — ask the user if not specified
- **ADO default project**: ask the user if not specified

## Step 0 — Verify Node.js

Run `node --version`. If it fails, stop and tell the user:

> Dev Mesh requires Node.js to run its MCP servers. Please install it from https://nodejs.org (LTS recommended), restart your terminal, then try again.

## Step 1 — Parse the input

The user passes one of:
- A bare ADO ID: `12345` or `#12345`
- An ADO URL: `https://dev.azure.com/<org>/<project>/_workitems/edit/12345`
- A Jira key: `PROJ-456` or `JIRA:PROJ-456`
- A Jira URL: `https://[org].atlassian.net/browse/NEXUS-456`

Detect source:
- Contains `dev.azure.com` or is a bare integer / `#N` pattern → **ADO**
- Contains `atlassian.net/browse` or matches `[A-Z]+-[0-9]+` → **Jira**

If source is ambiguous, ask the user: "Is this ticket in Azure DevOps or Jira?"

## Step 2 — Fetch ticket context

Delegate to the appropriate agent via the Task tool:

- **ADO**: Invoke `ado-reader` with `{ "work_item_id": <id> }`.
- **Jira**: Invoke `jira-reader` with `{ "issue_key": "<key>" }`.

Wait for the structured JSON context block before proceeding.

If the agent returns errors, surface them to the user clearly and stop.

## Step 3 — Produce the analysis

Render a structured report using this exact format:

```
# [Source] [Type] [ID]: [Title]

| Field | Value |
|---|---|
| **Source** | ADO / Jira |
| **Status** | [state] |
| **Assigned** | [assignee or "Unassigned"] |
| **Priority** | [priority] |
| **Sprint / Iteration** | [sprint or iteration path] |
| **Area / Project** | [area path or Jira project] |

---

## Acceptance Criteria

[Verbatim acceptance criteria from the ticket. If absent: "No acceptance criteria found in this ticket."]

---

## Description

[Ticket description — preserve structure if it uses headings or lists.]

---

## Child Items / Subtasks ([n])

| Type | ID | Title | Status | Assigned |
|---|---|---|---|---|

_(none)_ if empty.

---

## Related Items ([n])

| Type | ID | Title | Status |
|---|---|---|---|

_(none)_ if empty.

---

## Pull Requests ([n])

| PR | Title | Repo | Status | Branch | Date |
|---|---|---|---|---|---|

_(none)_ if empty.

---

## Summary

2–3 sentences: overall state of the ticket, how many child tasks are done vs open, PR/merge status, and any obvious blockers or outstanding work.
```

## Step 4 — Save the snapshot (optional)

If the user asked to save the analysis, write it to `analyses/[source]-[id]-analysis-YYYY-MM-DD.md`. Otherwise, output only to chat.
