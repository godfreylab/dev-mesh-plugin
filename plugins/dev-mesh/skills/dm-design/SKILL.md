---
description: >
  Generate a technical solution design document from an analyzed ADO or Jira ticket. Fetches the
  full ticket context, optionally searches Confluence for relevant spec pages, and produces a
  structured solution design document written to designs/. Use when a user says "design a solution
  for", "create a solution design for ticket", "write a tech design for", or "generate a design doc
  for [ticket ID or URL]". Also trigger when the user pastes a ticket ID and asks "how should we
  build this?" or "what's the technical approach?".
---

# dm-design

Generate a solution design document from an ADO or Jira ticket, enriched with Confluence context.

## Defaults

- **ADO organisation**: configured in `.mcp.json` — ask the user if not specified
- **ADO default project**: ask the user if not specified
- **Output path**: `designs/`

## Step 0 — Verify Node.js

Run `node --version`. If it fails, stop and tell the user Node.js is required.

## Step 1 — Parse the input

Accept the same input formats as `dm-analyze` (ADO ID, Jira key, or URL). Detect the source system using the same rules.

If the user also provides a Confluence space key or search term, note it for Step 3.

## Step 2 — Fetch ticket context

Delegate to `ado-reader` (ADO) or `jira-reader` (Jira) via the Task tool. Wait for the full structured context block.

If the agent returns errors, surface them and stop.

## Step 3 — Search Confluence for relevant docs

Invoke `confluence-reader` via the Task tool with:
```json
{
  "search_query": "[ticket title] OR [key technical terms from description]",
  "space_key": "[user-specified space or omit for global search]"
}
```

This is a best-effort search. If Confluence MCP authentication is not configured, warn the user that Confluence context will be absent from the design and continue.

## Step 4 — Clarification gate

Before generating the design:
1. Read the acceptance criteria from the ticket context.
2. If any requirement is ambiguous enough to produce materially different architectural choices, **stop and ask the user** targeted clarifying questions (numbered list). Do not generate the design until answered.
3. If all requirements are clear, state briefly "All acceptance criteria are clear — proceeding to design" and continue.

## Step 5 — Generate the solution design

Invoke `solution-designer` via the Task tool, passing:
```json
{
  "ticket": "<context from Step 2>",
  "confluence_pages": "<results from Step 3>",
  "options": {
    "output_path": "designs/"
  }
}
```

## Step 6 — Present results

Once `solution-designer` completes:
1. Tell the user the file path of the saved design document.
2. Output the Executive Summary and Requirements Summary sections to chat for an immediate overview.
3. Remind the user to review the Open Questions section if it is non-empty.
