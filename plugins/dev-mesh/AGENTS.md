# Dev Mesh — Agent Inventory

All agents in this plugin. Skills delegate to these agents via the Task tool.

| Agent | File | Role |
|---|---|---|
| `ado-reader` | `agents/ado-reader.md` | Reads ADO work items, PRs, iterations, and branches using ADO MCP tools |
| `jira-reader` | `agents/jira-reader.md` | Reads Jira issues, epics, stories, and sprints via Atlassian MCP |
| `confluence-reader` | `agents/confluence-reader.md` | Reads Confluence pages and spaces via Atlassian MCP |
| `solution-designer` | `agents/solution-designer.md` | Produces structured solution design documents from ticket context |
| `pr-reviewer` | `agents/pr-reviewer.md` | Reviews code changes using ticket context and Confluence specification docs |
| `impact-analyzer` | `agents/impact-analyzer.md` | Analyzes the blast radius of code changes across the system |

## Invocation pattern

Skills use the Task tool to delegate to these agents. The calling skill passes a structured JSON context block containing all fetched data so agents do not re-fetch the same MCP calls.
