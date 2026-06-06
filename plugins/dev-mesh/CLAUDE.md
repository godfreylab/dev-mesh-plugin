# Dev Mesh Plugin — Guidance for Claude

This file is loaded by Claude Code whenever the dev-mesh plugin is active. It applies to every skill and agent in this plugin.

## What this plugin is

Dev Mesh connects **Azure DevOps (ADO)**, **Jira**, and **Confluence** so Claude can analyze tickets, generate solution designs, review PRs, and surface relevant docs without switching tools.

## MCP tool namespaces

| Namespace prefix | System | Notes |
|---|---|---|
| `mcp__ado__*` | Azure DevOps | Org: configured in `.mcp.json` |
| `mcp__claude_ai_Atlassian__*` | Jira + Confluence | Authentication via Atlassian MCP |

Before making any MCP call, verify the relevant MCP server is reachable. If it fails, surface a clear blocker message — do not silently fall back to ad-hoc web browsing.

## Node.js prerequisite

Both MCP servers run via `npx`. If `node --version` fails, stop and instruct the user to install Node.js from https://nodejs.org (LTS recommended) and restart their terminal.

## Security and data handling

- Never output, log, or store credentials, API tokens, PII, or payment card data.
- All auth/input-handling logic must include inline comments explaining the security rationale.
- Apply OWASP Top 10 considerations to any code generated during a PR review or solution design.
- If ticket content contains sensitive fields (e.g., names, account numbers), refer to them generically in outputs.

## Defaults

- **ADO organisation**: set in `.mcp.json` — ask the user if not clear from context
- **ADO default project**: ask the user if not specified
- **Confluence default space**: ask the user if not clear from context

## Cross-system linking

When a ticket in one system references an ID from another (e.g., an ADO work item linked to a Jira issue), always fetch both sides before producing output. Partial context produces incomplete designs and missed review findings.

## Output directories

Skills write output to the directories listed below. Create them if they do not exist.

| Output type | Directory |
|---|---|
| Ticket analysis snapshots | `analyses/` |
| Solution design docs | `designs/` |
| PR review reports | `reviews/` |
| Standup summaries | `standups/` |
| Sync reports | `sync-reports/` |
