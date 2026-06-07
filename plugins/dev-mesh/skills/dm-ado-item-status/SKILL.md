---
description: >
  Retrieve the complete status of an Azure DevOps User Story, including story details, acceptance
  criteria, story points, assignee, parent item, related items, linked pull requests, branches, and
  commits. Produces a comprehensive status document saved to docs/. Use when the user shares an ADO
  User Story ID (e.g., "123", "#123", "story 456") or an ADO work item URL and asks for its status,
  linked work, PR/branch/commit activity, or wants a full delivery picture. Phrases like "status of
  story 123", "what's the status on ADO 456", "show me everything for user story 789", "give me the
  full picture for this story", or "dm-ado-item-status 123" all trigger this skill. Requires a live
  ADO MCP connection.
---

# dm-ado-item-status

Fetch the complete status of an Azure DevOps User Story — story details, parent, related items,
PRs, branches, and commits — and write a status document to `docs/`.

## Defaults

- **ADO organisation**: read from the `ADO_ORG` environment variable (set in
  `.claude/settings.local.json` under `env.ADO_ORG`) — ask the user if not set
- **ADO project**: ask the user if not specified
- **Output directory**: `docs/`

## Step 0 — Verify ADO MCP connection

Run `node --version`. If it fails, stop and tell the user:

> Dev Mesh requires Node.js to run its MCP servers. Please install it from https://nodejs.org
> (LTS recommended), restart your terminal, then try again.

Read the ADO organisation from the `ADO_ORG` environment variable. If it is empty or unset, stop
and tell the user:

> `ADO_ORG` is not configured. Copy `.claude/settings.local.json.example` to
> `.claude/settings.local.json` and set `ADO_ORG` to your Azure DevOps organisation name.

Then verify the ADO MCP server is reachable by calling `mcp__ado__core_list_projects` with
`{ "organization": "<ADO_ORG>" }`. If it fails, stop and tell the user:

> Cannot reach the Azure DevOps MCP server. Ensure your `AZURE_DEVOPS_EXT_PAT` environment
> variable is set and holds a token with scopes: Work Items (Read), Code (Read), Pull Request
> Threads (Read). Restart your terminal after setting it.

## Step 1 — Parse the input

The user passes one of:
- A bare integer: `123` or `#123`
- An ADO work item URL: `https://dev.azure.com/<org>/<project>/_workitems/edit/123`

Extract the numeric work item ID. If a URL is provided, also extract the org and project — these
override the defaults above.

If the input is ambiguous (e.g., the number could be a Jira ticket), ask: "Is this a ticket in
Azure DevOps?"

## Step 2 — Fetch the User Story

Call `mcp__ado__wit_get_work_item`:
```json
{
  "organization": "<org>",
  "id": <work_item_id>,
  "$expand": "all"
}
```

Extract these fields from the response:
- `fields["System.Title"]` — title
- `fields["System.WorkItemType"]` — work item type
- `fields["System.State"]` — current state
- `fields["System.AssignedTo"]["displayName"]` — assignee
- `fields["System.Description"]` — description (may contain HTML — strip tags for display)
- `fields["Microsoft.VSTS.Common.AcceptanceCriteria"]` — acceptance criteria (strip HTML tags)
- `fields["Microsoft.VSTS.Scheduling.StoryPoints"]` — story points
- `fields["System.IterationPath"]` — sprint / iteration
- `fields["System.AreaPath"]` — area path
- `fields["System.Tags"]` — tags
- `relations` — array of all linked items (parent, related, PR, branch, commit artifact links)

If the work item is not found or is not a User Story type, inform the user and stop.

## Step 3 — Fetch the parent item

Scan `relations` for an entry where `rel == "System.LinkTypes.Hierarchy-Reverse"`. This is the
parent (e.g., an Epic or Feature).

If found:
- Extract the parent work item ID from the `url` field (last path segment is the ID)
- Call `mcp__ado__wit_get_work_item` for the parent with `{ "$expand": "fields" }`
- Record: type, ID, title, state, assignee

If no parent relation exists, record: _No parent item._

## Step 4 — Fetch related items

Scan `relations` for entries where `rel` is one of:
- `"System.LinkTypes.Related"` — general related items
- `"System.LinkTypes.Dependency-Forward"` — successor dependencies
- `"System.LinkTypes.Dependency-Reverse"` — predecessor dependencies
- `"System.LinkTypes.Hierarchy-Forward"` — child items

Collect all work item IDs from the `url` fields. If any exist, call
`mcp__ado__wit_get_work_items_batch_by_ids` in a single batch:
```json
{
  "organization": "<org>",
  "ids": [<id1>, <id2>, ...]
}
```

Record each item's: type, ID, title, state, assignee, and link type (e.g., "Related",
"Child", "Successor").

If none, record: _No related items._

## Step 5 — Fetch linked pull requests

Scan `relations` for entries where `rel == "ArtifactLink"` and the `url` value contains
`/PullRequestId/`. The artifact URL encodes the project ID, repo ID, and PR ID — parse the PR ID
from the last path segment (URL-decoded).

For each PR, call `mcp__ado__repo_get_pull_request_by_id`:
```json
{
  "organization": "<org>",
  "project": "<project>",
  "repositoryId": "<repo-id-from-artifact-url>",
  "pullRequestId": <pr_id>
}
```

Record each PR's: ID, title, repository name, status (Active / Completed / Abandoned), source
branch, target branch, created by (author), created date, and last updated date.

If no PR artifact links are found in relations, record: _No linked pull requests._

## Step 6 — Resolve branches

From each PR fetched in Step 5, record the `sourceRefName` (source branch). Strip the
`refs/heads/` prefix for display.

Additionally, scan `relations` for entries where `rel == "ArtifactLink"` and the `url` value
contains `/Ref/`. Parse the branch name from the artifact URL and record it alongside its
repository name.

Deduplicate. If no branches are found, record: _No branches found._

## Step 7 — Fetch commits

Scan `relations` for entries where `rel == "ArtifactLink"` and the `url` value contains
`/Commit/`. Parse the commit SHA from the artifact URL.

For each distinct commit SHA, call `mcp__ado__repo_search_commits`:
```json
{
  "organization": "<org>",
  "project": "<project>",
  "searchCriteria": {
    "ids": ["<commit-sha>"]
  }
}
```

Record each commit's: short SHA (first 8 chars), author name, committed date, and commit message
(first line only — truncate at 72 characters).

If no commit artifact links are found, record: _No commits found._

## Step 8 — Compose the status document

Use this exact structure:

```markdown
# ADO [ID]: [Title]

> **Status Snapshot** — generated [YYYY-MM-DD]

---

## Story Details

| Field | Value |
|---|---|
| **ID** | [id] |
| **Type** | [work item type] |
| **State** | [state] |
| **Assigned To** | [assignee or "Unassigned"] |
| **Story Points** | [points or "—"] |
| **Sprint / Iteration** | [iteration path] |
| **Area** | [area path] |
| **Tags** | [tags or "—"] |

---

## Acceptance Criteria

[Verbatim acceptance criteria — preserve list structure. If absent: "No acceptance criteria found."]

---

## Description

[Story description — preserve headings and lists. If absent: "No description provided."]

---

## Parent Item

| Field | Value |
|---|---|
| **Type** | [type] |
| **ID** | [id] |
| **Title** | [title] |
| **State** | [state] |
| **Assigned To** | [assignee or "Unassigned"] |

_(none)_ if no parent.

---

## Related Items ([n])

| Type | ID | Title | State | Assigned | Link Type |
|---|---|---|---|---|---|

_(none)_ if empty.

---

## Pull Requests ([n])

| PR # | Title | Repo | Status | Source Branch | Target Branch | Author | Date |
|---|---|---|---|---|---|---|---|

_(none)_ if empty.

---

## Branches ([n])

| Branch | Repo |
|---|---|

_(none)_ if empty.

---

## Commits ([n])

| SHA | Author | Date | Message |
|---|---|---|---|

_(none)_ if empty.

---

## Summary

[3–5 sentences covering: current delivery state, acceptance criteria completeness, PR / branch /
commit activity, open related work, and any visible blockers or risks.]
```

## Step 9 — Save the document

Sanitize the title for the folder name: lowercase, replace spaces and non-alphanumeric characters
with hyphens, collapse consecutive hyphens, and truncate to 60 characters.

Create the folder:
```
docs/[id]-[sanitized-title]/
```

Append the following footer to the document before writing (replace `[YYYY-MM-DD]` with today's
date):

```markdown
---

_Last updated: [YYYY-MM-DD]_
```

Write the document to:
```
docs/[id]-[sanitized-title]/[id]-status.md
```

Create the `docs/` directory and the story subfolder if they do not exist.

## Step 10 — Present to chat

Output the **Story Details** table and the **Summary** section to chat for immediate visibility.
Tell the user: `Full status document saved to docs/[id]-[sanitized-title]/[id]-status.md`
