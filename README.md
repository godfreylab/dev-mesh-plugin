# dev-mesh-plugin

A Claude Code plugin marketplace for Dev Mesh — AI-powered integration across Azure DevOps, Jira, and Confluence.

## What ships

| Plugin | Description |
|---|---|
| `dev-mesh` | Ticket analysis, solution design generation, and intelligent PR review |

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
/dm-analyze ADO:12345
/dm-analyze NEXUS-456

/dm-ado-item-status 12345
/dm-ado-item-status https://dev.azure.com/<your-org>/<your-project>/_workitems/edit/12345

/dm-design ADO:12345

/dm-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789

/dm-ado-pr-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789

/dm-standup

/dm-sync ADO:12345

/dm-confluence-search "retry logic"
```

### Skills

| Skill | What it does |
|---|---|
| `/dm-analyze` | Fetch and analyze a ticket from ADO or Jira — title, acceptance criteria, child tasks, linked PRs, and a summary |
| `/dm-ado-item-status` | Full status snapshot of an ADO User Story: story details, parent, related items, PRs, branches, and commits. Saves to `docs/[id]-[title]/[id]-status.md` |
| `/dm-design` | Generate a solution design document from an ADO or Jira ticket |
| `/dm-review` | Review a pull request with full ticket and Confluence context |
| `/dm-ado-pr-review` | AI code review on an ADO PR — posts inline comments directly on the PR |
| `/dm-standup` | Summarize your current sprint activity and PR status for standup |
| `/dm-sync` | Cross-reference an ADO work item against its Jira counterpart |
| `/dm-confluence-search` | Search Confluence for documentation related to current work |

## Prerequisites

| Tool | Install |
|---|---|
| Node.js (LTS) | https://nodejs.org |
| Python + `uv` | https://docs.astral.sh/uv/getting-started/installation/ |

## MCP setup

This plugin requires the following MCP servers. Core servers are defined in `.mcp.json`; Microsoft 365 integrations are optional add-ons.

### Core servers (`.mcp.json`)

| Server | Package | Purpose | Source |
|---|---|---|---|
| `ado` | `@azure-devops/mcp` (npx) | Azure DevOps — work items, PRs, pipelines | [GitHub](https://github.com/microsoft/azure-devops-mcp) |
| `mcp-atlassian` | `mcp-atlassian` (uvx) | Jira + Confluence | [GitHub](https://github.com/sooperset/mcp-atlassian) |
| `CLI-Microsoft365` | `@pnp/cli-microsoft365-mcp-server` (npx) | Microsoft 365 CLI — run any `m365` command against Teams, SharePoint, Exchange, and more | [GitHub](https://github.com/pnp/cli-microsoft365-mcp-server) |

### claude.ai remote integrations (optional)

These are remote MCP servers provided by claude.ai. Enable them in **claude.ai → Settings → Integrations** — no local install required.

| Integration | Tools provided | Purpose |
|---|---|---|
| **Microsoft 365** | `chat_message_search`, `outlook_email_search`, `sharepoint_search`, `outlook_calendar_search`, `find_meeting_availability` | Search Teams chats, Outlook mail/calendar, and SharePoint — read-only, uses your logged-in M365 identity |
| **Microsoft Learn** | `microsoft_docs_search`, `microsoft_code_sample_search`, `microsoft_docs_fetch` | Search and fetch official Microsoft / Azure documentation and code samples |

### Configuration

Copy `.claude/settings.local.json.example` to `.claude/settings.local.json` and fill in your credentials:

```jsonc
// .claude/settings.local.json
{
  "env": {
    "ADO_ORG": "your-ado-org",                         // ADO organisation slug
    "JIRA_URL": "https://your-org.atlassian.net",
    "JIRA_USERNAME": "you@example.com",
    "JIRA_API_TOKEN": "<Atlassian API token>",          // https://id.atlassian.com/manage-profile/security/api-tokens
    "CONFLUENCE_URL": "https://your-org.atlassian.net/wiki",
    "CONFLUENCE_USERNAME": "you@example.com",
    "CONFLUENCE_API_TOKEN": "<Atlassian API token>"
  },
  "enabledMcpjsonServers": ["ado", "mcp-atlassian"]
}
```

`settings.local.json` is gitignored — never commit real tokens.

### Verify both servers loaded

After restarting Claude Code run `/mcp` — both `ado` and `mcp-atlassian` should show as connected.

## Structure

See `plugins/dev-mesh/README.md` for the full component inventory.
See `CLAUDE.md` for contribution conventions.
