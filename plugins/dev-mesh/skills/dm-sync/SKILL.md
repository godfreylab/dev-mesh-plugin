---
description: >
  Cross-reference and sync information between Azure DevOps and Jira for a given ticket or
  work item. Identifies inconsistencies in status, title, assignee, or linked items between
  the two systems. Use when the user asks "sync this ticket", "compare ADO and Jira for",
  "is ADO in sync with Jira for [ID]", or "what's different between these tickets?". Also
  trigger when the user provides both an ADO ID and a Jira key and asks to reconcile them.
---

# dm-sync

Cross-reference ADO and Jira (and optionally Confluence) for a work item, surfacing inconsistencies.

## Defaults

- **ADO organisation**: configured in `.mcp.json` — ask the user if not specified
- **ADO default project**: ask the user if not specified

## Step 0 — Verify Node.js

Run `node --version`. If it fails, stop and tell the user Node.js is required.

## Step 1 — Parse the input

The user provides one or both of:
- An ADO work item ID
- A Jira issue key

If only one is provided, check the ticket's description and linked items for a reference to its counterpart in the other system. If none is found, ask the user for the matching ID.

## Step 2 — Fetch both sides concurrently

Invoke `ado-reader` and `jira-reader` concurrently via the Task tool (two parallel Task calls). Wait for both to complete before proceeding.

If either agent returns errors, surface them and note that the sync comparison will be partial.

## Step 3 — Compare and reconcile

Compare the two context blocks across:

| Field | ADO field | Jira field |
|---|---|---|
| Title / Summary | `title` | `summary` |
| Status | `state` | `status` |
| Assignee | `assigned_to` | `assignee` |
| Priority | `priority` | `priority` |
| Acceptance criteria | `acceptance_criteria` | `acceptance_criteria` |
| Sprint / Iteration | `iteration_path` | `sprint` |

For each field, note: Matched / Mismatch / Missing on one side.

## Step 4 — Check Confluence references

If either ticket references a Confluence page in its description, fetch those pages via `confluence-reader` and note whether the implementation described matches both tickets' acceptance criteria.

## Step 5 — Render the sync report

```
# Sync Report: ADO:[ID] ↔ Jira:[KEY]

**Generated:** [YYYY-MM-DD]

## Field Comparison

| Field | ADO | Jira | Status |
|---|---|---|---|
| Title | [ADO title] | [Jira summary] | Matched / Mismatch |
| Status | [ADO state] | [Jira status] | Matched / Mismatch |
| Assignee | [name] | [name] | Matched / Mismatch |
| Priority | [n] | [p] | Matched / Mismatch |
| Sprint | [path] | [sprint name] | Matched / Mismatch |

## Acceptance Criteria Comparison

[Side-by-side of ADO acceptance criteria vs Jira acceptance criteria. Note any gaps or contradictions.]

## Linked Items

| System | Linked Items |
|---|---|
| ADO | [child tasks, PRs] |
| Jira | [subtasks, linked issues] |

## Inconsistencies Found ([n])

[Numbered list of each mismatch. For each: what the discrepancy is, which system appears to be authoritative (based on recency or ticket state), and a recommended action.]

## Recommendations

[Actionable list of what to update and in which system to bring them into alignment.]
```

Save the report to `sync-reports/ado-[ID]-jira-[KEY]-sync-YYYY-MM-DD.md` if the user requests it, otherwise output to chat only.
