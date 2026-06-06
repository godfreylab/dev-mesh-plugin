---
description: >
  Summarize recent work items and PR activity for a standup update. Fetches the current sprint's
  in-progress and recently completed items assigned to the user, plus any open or recently merged
  PRs. Produces a concise standup-ready summary. Use when the user says "standup", "what did I
  work on", "generate my standup notes", "what's my status for today", or "summarize my sprint
  activity". Defaults to the current iteration and current user.
---

# dm-standup

Generate a standup summary from ADO sprint activity and PR status.

## Defaults

- **ADO organisation**: configured in `.mcp.json` — ask the user if not specified
- **ADO default project**: ask the user if not specified
- **Lookback window**: last 24 hours for "done yesterday"; current sprint for "in progress / blocked"

## Step 0 — Verify Node.js

Run `node --version`. If it fails, stop and tell the user Node.js is required.

## Step 1 — Identify the user

The standup is for the currently authenticated user. If the user specifies a name or email, use that instead.

## Step 2 — Fetch current sprint items

Invoke `ado-reader` via the Task tool with:
```json
{
  "iteration_path": "[current sprint path]"
}
```

Filter the results to items assigned to the current user. Categorise by state:
- **Active / In Progress** — items the user is actively working on
- **Resolved / Closed / Done (within last 24 hours)** — completed yesterday
- **Blocked** — items with a blocked tag or state

## Step 3 — Fetch open PRs

Call `mcp__ado__repo_list_pull_requests_by_repo_or_project` filtered by author = current user and status = `active`. Also fetch PRs merged in the last 24 hours.

## Step 4 — Render the standup summary

```
## Standup — [YYYY-MM-DD]

### Yesterday
[Bullet list of resolved/closed items and merged PRs from the last 24h.
If nothing: "No items completed in the last 24 hours."]

### Today
[Bullet list of active/in-progress items and open PRs.
If nothing: "No active items in current sprint."]

### Blockers
[Bullet list of blocked items.
If nothing: "No blockers."]

### Notes
[Any PRs awaiting review, items nearing sprint deadline, or other flags worth mentioning.]
```

Keep the standup summary concise — one line per item. Do not include full descriptions.
