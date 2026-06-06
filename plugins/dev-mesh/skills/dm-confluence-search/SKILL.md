---
description: >
  Search Confluence for pages and documentation relevant to the current work. Returns the top
  matching pages with titles, excerpts, and URLs. Use when the user asks "find Confluence docs
  for", "search Confluence for", "what does the spec say about", "is there documentation on",
  or "find the ADR for [topic]". Also trigger when the user is about to start on a ticket and
  wants to know what existing documentation exists for that feature area.
---

# dm-confluence-search

Search Confluence and surface relevant docs for current work.

## Step 0 — Verify Node.js

Run `node --version`. If it fails, stop and tell the user Node.js is required.

## Step 1 — Parse the input

The user provides:
- A free-text search query (e.g., `"payment gateway retry logic"`)
- Optionally a Confluence space key to scope the search
- Optionally a ticket ID to derive the search terms from (in which case, fetch the ticket title and key terms first)

If the user provides a ticket ID instead of a search query, call `ado-reader` or `jira-reader` first to extract the ticket title and acceptance criteria keywords.

## Step 2 — Authenticate to Confluence

If not already authenticated, call `mcp__claude_ai_Atlassian__authenticate`. If authentication fails, stop with a clear message:

> Confluence search requires Atlassian MCP authentication. Run `mcp__claude_ai_Atlassian__authenticate` to configure access.

## Step 3 — Search Confluence

Invoke `confluence-reader` via the Task tool with:
```json
{
  "search_query": "<derived or user-supplied query>",
  "space_key": "<user-supplied space key or omit>"
}
```

## Step 4 — Present results

```
## Confluence Search: "[query]"

**Space:** [space key or "All spaces"] | **Date:** [YYYY-MM-DD]

### Results ([n] pages found)

---

**1. [Page Title]**
Space: [space key] | Last modified: [date]
URL: [page URL]

> [Excerpt — first 200 characters of body]

---

**2. [Page Title]**
...

---

[If no results: "No Confluence pages found for this query. Try broadening your search terms or checking the space key."]
```

If the user asked for full page content (e.g., "show me the full spec for X"), fetch and display the full `body_markdown` for the top matching page.
