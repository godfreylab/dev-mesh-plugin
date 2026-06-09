# dev-mesh-plugin

A Claude Code plugin for Dev Mesh — AI-powered integration with Azure DevOps.

## Installation

### Claude Code

Run these two commands inside Claude Code:

```
/plugin marketplace add godfreylab/dev-mesh-plugin
/plugin install dev-mesh
```

Then restart Claude Code and run `/mcp` to verify the MCP servers are connected.

## Quick start

```
/dm-ado-item-status 12345
/dm-ado-item-status https://dev.azure.com/<your-org>/<your-project>/_workitems/edit/12345

/dm-ado-pr-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789

/dm-analyze 12345
/dm-analyze https://dev.azure.com/<your-org>/<your-project>/_workitems/edit/12345
```

### Skills

| Skill | What it does |
|---|---|
| `/dm-ado-item-status` | Full status snapshot of an ADO User Story: story details, parent, related items, PRs, branches, and commits. Saves to `docs/`. |
| `/dm-ado-pr-review` | AI code review on an ADO PR — posts inline comments directly on the PR. Saves a report to `reviews/`. |
| `/dm-analyze` | End-to-end solution design for an ADO User Story. Scans linked repos, Jira issues, and Confluence pages; produces a story breakdown, API contracts, DB migration plan, and workflow diagrams. Saves to `docs/`. |

#### `/dm-analyze` in detail

`/dm-analyze <id-or-url>` turns a User Story into a ready-to-implement technical design:

1. **Loads story context** — fetches (or reuses a cached) status snapshot from `dm-ado-item-status`, including description, acceptance criteria, parent Epic/Feature, sprint, and all linked items.
2. **Fetches cross-system context** — resolves any Jira issue keys and Confluence page URLs found in the story, pulling in constraints, architecture decisions, and existing specs.
3. **Scans the real codebase** — searches each involved ADO repo for relevant controllers, domain models, migration files, and tests; reads key files so design decisions are grounded in actual class names, table names, and project conventions.
4. **Evaluates approaches** — for the most ambiguous design decision, proposes 2–3 concrete options with pros/cons and a codebase-backed recommendation.
5. **Produces a full solution design** saved to `docs/[id]-[title]/[id]-analyze.md` covering:
   - Problem statement and AC review (with testability and codebase impact per criterion)
   - Story breakdown — child stories with draft ACs, point estimates, sequencing dependencies, and exact files to touch, all anchored to the same sprint and Epic/Feature as the original
   - Per-repo file change plan (files to create, modify, delete — with real paths and class names)
   - Database migration DDL and zero-downtime safety classification
   - Full API contracts — request/response JSON schemas, auth scopes, validation rules, all error codes
   - Event/message payload schemas (if the project uses async messaging)
   - High-level architecture diagram and detailed request-sequence diagram (Mermaid)
   - Risks & dependencies table and open questions

## Prerequisites

| Tool | Install |
|---|---|
| Node.js (LTS) | https://nodejs.org |
| Python + `uv` | https://docs.astral.sh/uv/getting-started/installation/ |

## MCP setup

MCP servers are defined in `.mcp.json`.

| Server | Package | Purpose |
|---|---|---|
| `ado` | `@azure-devops/mcp` (npx) | Azure DevOps — work items, PRs, pipelines |
| `mcp-atlassian` | `mcp-atlassian` (uvx) | Jira + Confluence |
| `CLI-Microsoft365` | `@pnp/cli-microsoft365-mcp-server` (npx) | Microsoft 365 CLI |

### Configuration

Copy `.claude/settings.local.json.example` to `.claude/settings.local.json` and fill in your credentials:

```jsonc
{
  "env": {
    "ADO_ORG": "your-ado-org",
    "JIRA_URL": "https://your-org.atlassian.net",
    "JIRA_USERNAME": "you@example.com",
    "JIRA_API_TOKEN": "<your-atlassian-api-token>",
    "CONFLUENCE_URL": "https://your-org.atlassian.net/wiki",
    "CONFLUENCE_USERNAME": "you@example.com",
    "CONFLUENCE_API_TOKEN": "<your-atlassian-api-token>"
  }
}
```

`settings.local.json` is gitignored — never commit real tokens.

### Verify the server loaded

After restarting Claude Code run `/mcp` — `ado` should show as connected.

## Structure

See `plugins/dev-mesh/README.md` for the full component inventory.
See `CLAUDE.md` for contribution conventions.
