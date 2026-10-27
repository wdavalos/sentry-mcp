# Sentry Error Events and Issues — MCP Server

Sentry Error Monitoring Data for AI Agents. This Actor wraps the Sentry CLI as an MCP server, giving AI agents programmatic access to error events, issues, releases, and project statistics across your Sentry workspace.

Sentry MCP exposes 219 tools that AI agents can call directly to monitor errors, investigate stack traces, track bug fixes, and query deployment data — without leaving the agent workflow. Connect via MCP protocol or REST API.

## Features

- **Error Event Search**: Query events by project, time range, severity, and custom filters using Sentry search syntax
- **Issue Management**: Create, update, assign, resolve, and archive issues directly from AI agent workflows
- **Release Tracking**: Query deployment data, release commits, and associated error signatures
- **Project Statistics**: Fetch error rates, user impact counts, and trend data per project
- **Breadcrumb and Context Access**: Retrieve full stack traces, local variables, and execution breadcrumbs
- **Organization and Team Data**: List organizations, teams, and member access across your Sentry workspace
- **Event Aggregation**: Run aggregate queries across event clusters for pattern detection
- **Cursor-based Pagination**: Handle large result sets with efficient pagination
- **MCP Protocol Support**: Serves the MCP JSON-RPC standard for AI agent integration
- **Pay-Per-Event Pricing**: Apify PPE billing with per-tool pricing breakdown

## Quick Start

### MCP Protocol (Recommended for AI Agents)

```bash
# Connect to the MCP endpoint
https://api.apify.com/v2/acts/YOUR-USER-NAME~sentry-mcp/mcp

# Set SENTRY_AUTH_TOKEN as an Apify secret before connecting
```

### REST API Direct Call

```bash
curl -X POST https://api.apify.com/v2/acts/YOUR-USER-NAME~sentry-mcp/runs \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "sentry_events_list",
    "args": {
      "project": "my-backend-service",
      "first": 10,
      "query": "is:unresolved level:error"
    }
  }'
```

### Authentication Setup

1. Generate a Sentry API token at `https://sentry.io/settings/account/api/tokens/`
2. Required scopes: `event:read`, `project:read`, `org:read`, `release:read`
3. Add `SENTRY_AUTH_TOKEN` as a secret in Apify Console > Actor > Secrets
4. The token is never logged or exposed in output

## Input

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tool` | string | Yes | Tool name (e.g., `sentry_events_list`, `sentry_issues_list`) |
| `args` | object | No | Tool arguments as key-value pairs. See tool documentation for supported parameters. |
| `timeout` | number | No | Per-call timeout in seconds (default: 60) |

**Example input:**
```json
{
  "tool": "sentry_issues_list",
  "args": {
    "project": "my-backend-service",
    "first": 25,
    "query": "is:unresolved level:error",
    "sort": "priority",
    "statsPeriod": "14d"
  }
}
```

**Search syntax examples:**
- `is:unresolved level:error` — All unresolved errors
- `browser:Chrome` — Errors from Chrome browser
- `release:1.2.3` — Errors from a specific release
- `user.email:admin@example.com` — Errors affecting a specific user

## Tools

### Events

| Tool | Description | Price |
|------|-------------|-------|
| `sentry_events_list` | List events for a project with optional filters | $0.10 |
| `sentry_events_aggregate` | Aggregate events by field (count, unique count, p75) | $0.10 |
| `sentry_event_details` | Get full event details including stack trace | $0.08 |
| `sentry_event_tags` | Get event tag values | $0.05 |

### Issues

| Tool | Description | Price |
|------|-------------|-------|
| `sentry_issues_list` | List issues with filter and sort options | $0.08 |
| `sentry_issues_create` | Create a new issue | $0.05 |
| `sentry_issues_update` | Update issue (status, assignee, attributes) | $0.05 |
| `sentry_issues_delete` | Resolve or delete an issue | $0.03 |
| `sentry_issue_activity` | Get issue activity stream and comments | $0.05 |
| `sentry_issue_todos` | List and manage issue action items | $0.05 |

### Releases

| Tool | Description | Price |
|------|-------------|-------|
| `sentry_releases_list` | List releases for a project | $0.05 |
| `sentry_releases_create` | Create a new release | $0.05 |
| `sentry_releases_deploys` | List deploys for a release | $0.03 |
| `sentry_releases_commits` | Get commits associated with a release | $0.05 |

### Projects and Teams

| Tool | Description | Price |
|------|-------------|-------|
| `sentry_projects_list` | List all projects | $0.05 |
| `sentry_teams_list` | List organization teams | $0.03 |
| `sentry_org_list` | List organizations | $0.03 |
| `sentry_project_stats` | Get project-level error rates and trends | $0.05 |

### Discovery

| Tool | Description | Price |
|------|-------------|-------|
| `list_tools` | List all available Sentry tools (free) | $0.00 |

## Output

Results are written to the default dataset in JSON format.

**Example output:**
```json
{
  "result": {
    "data": [
      {
        "id": "1234567890abcdef",
        "type": "error",
        "message": "TypeError: Cannot read property 'foo' of undefined",
        "culprit": "src/handlers/user.js in processUser",
        "project": "my-backend-service",
        "platform": "javascript",
        "count": 42,
        "userCount": 5,
        "firstSeen": "2026-05-15T10:30:00Z",
        "lastSeen": "2026-05-19T14:22:00Z"
      }
    ],
    "meta": {
      "next": "https://api.sentry.io/api/0/projects/...",
      "total": 147
    }
  },
  "tool": "sentry_events_list",
  "timestamp": "2026-05-19T20:00:00.000Z",
  "billing": {
    "eventType": "sentry_events_list",
    "unitPrice": 0.10,
    "units": 1
  }
}
```

Download output in JSON, CSV, HTML, or Excel from the Apify Console dataset view.

## Pricing

All pricing is Pay-Per-Event (PPE). You control costs by setting tool-specific prices or enabling/disabling tools in the Apify Console.

| Event Type | Price | Notes |
|------------|-------|-------|
| `list_tools` | $0.00 | Free discovery endpoint |
| `sentry_events_list` | $0.10 | Primary read operation |
| `sentry_events_aggregate` | $0.10 | Analytics queries |
| `sentry_event_details` | $0.08 | Detail fetch |
| `sentry_issues_list` | $0.08 | Issue queries |
| `sentry_issues_create` | $0.05 | Write operations |
| `sentry_issues_update` | $0.05 | Modifications |
| `sentry_issues_delete` | $0.03 | Deletions |
| `sentry_releases_list` | $0.05 | Release queries |
| `sentry_projects_list` | $0.05 | Project listing |
| `sentry_project_stats` | $0.05 | Statistics |
| Other tools | $0.03 – $0.05 | Varies by complexity |

**Cost estimation**: A typical bug investigation workflow (3 events_list + 1 issue_activity) costs approximately $0.35 per full stack trace retrieval.

Default events (actor start, heartbeat) are free. Set pricing per-tool in Apify Console > Actor > Pricing.

## Troubleshooting

**"Authentication failed" error**
- Verify `SENTRY_AUTH_TOKEN` is set as an Apify secret
- Check token scopes: needs `event:read`, `project:read`, `org:read` minimum
- Tokens expire — regenerate at `sentry.io/settings/account/api/tokens/` if needed

**"Project not found" response**
- Ensure project slug matches exact Sentry project name (case-sensitive)
- Confirm the token has access to the project organization
- Some enterprise Sentry instances use custom project paths

**Rate limit errors (429)**
- Sentry API limits apply per token, typically 500 requests/minute
- Implement exponential backoff: `retry-after: seconds * 2`
- Consider batching multiple queries into single calls where possible

**Empty results with no error**
- Verify the query syntax is correct (Sentry uses `key:value`, not `key=value`)
- Check that events exist in the selected time window (`statsPeriod` defaults to 14d)
- Ensure the project has events matching your filters

**MCP client connection refused**
- The actor serves MCP on the standby port at `/mcp` endpoint
- Verify the actor is running (not just built) — use `apify run` for local testing
- Check firewall allows outbound port 443 to `api.apify.com`

**Actor times out on large queries**
- Reduce `first` parameter (default may be high)
- Add time-bound filters: `age:>1h` or `statsPeriod:24h`
- Set higher `timeout` in input for expensive operations (stack trace retrieval)

---

Built on [Apify](https://apify.com) — the platform for web scraping, data extraction, and browser automation.