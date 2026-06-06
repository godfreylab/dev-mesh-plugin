---
description: >
  Review a pull request using context from linked ADO or Jira tickets and relevant Confluence
  specification docs. Produces a structured review report in reviews/. Use when the user shares a
  PR URL or PR number and asks for a review, says "review this PR", "check this pull request", or
  "review PR [N]". Also trigger when the user asks "does this PR match the ticket?" or "is this
  implementation correct for [ticket ID]?". Always prefer this skill over ad-hoc code review for
  any PR that has linked work items.
---

# dm-review

Review a PR with full ticket context and Confluence specification docs.

## Defaults

- **ADO organisation**: configured in `.mcp.json` — ask the user if not specified
- **ADO default project**: ask the user if not specified
- **Output path**: `reviews/`

## Step 0 — Verify Node.js

Run `node --version`. If it fails, stop and tell the user Node.js is required.

## Step 1 — Parse the input

Accept:
- A bare PR number: `789`
- An ADO PR URL: `https://dev.azure.com/<org>/<project>/_git/<repo>/pullrequest/789`
- A repo name + PR number: `repo-name#789`

Extract the integer PR ID and repository name. If the repository name is not clear, call `mcp__ado__repo_list_repos_by_project` and ask the user to confirm which repo.

## Step 2 — Fetch PR and linked ticket context

Invoke `ado-reader` via the Task tool with:
```json
{
  "pr_url": "<resolved PR URL>"
}
```

The agent returns the PR metadata, diff summary, and hydrated linked tickets (with acceptance criteria).

If the PR has no linked work items, warn the user:

> This PR has no linked work items. The review will cover code quality and security only — acceptance criteria coverage cannot be assessed without linked tickets.

## Step 3 — Fetch Confluence context

For each linked ticket, search Confluence using `confluence-reader` for pages relevant to the ticket's title and key technical terms. Pass the union of all search results to the reviewer.

If Confluence MCP is not authenticated, skip this step and note the absence in the review report.

## Step 4 — Assess impact

Invoke `impact-analyzer` via the Task tool with:
```json
{
  "pr": "<pr context from Step 2>",
  "tickets": "<linked tickets from Step 2>",
  "options": { "depth": "shallow" }
}
```

## Step 5 — Generate the review

Invoke `pr-reviewer` via the Task tool, passing the full assembled context:
```json
{
  "pr": "<from Step 2>",
  "tickets": "<from Step 2>",
  "confluence_pages": "<from Step 3>",
  "impact": "<from Step 4>",
  "options": {
    "output_path": "reviews/"
  }
}
```

## Step 6 — Present results

Once `pr-reviewer` completes:
1. Tell the user the file path of the saved review report.
2. Output the Summary, Ticket Alignment table, and Recommendation to chat immediately.
3. If any Critical or Major issues were found, list them in chat so the user sees them without reading the full report.
4. Surface any `[HUMAN REVIEW RECOMMENDED]` items explicitly.
