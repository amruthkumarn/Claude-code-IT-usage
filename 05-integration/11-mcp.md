# Section 11: Model Context Protocol (MCP)

## Table of Contents
1. [What is MCP?](#what-is-mcp)
2. [MCP Server Types](#mcp-server-types)
3. [Installing MCP Servers](#installing-mcp-servers)
4. [Banking IT Integrations](#banking-it-integrations)
5. [Best Practices](#best-practices)

---

## What is MCP?

**Model Context Protocol (MCP)** allows Claude Code to connect to external tools and services, extending its capabilities beyond local file operations.

### Common Integrations

- **Databases**: PostgreSQL, MySQL, MongoDB
- **Issue Trackers**: Jira, Linear, GitHub Issues
- **Monitoring**: Datadog, Splunk, Prometheus
- **APIs**: Internal REST/GraphQL APIs
- **Cloud Services**: AWS, Google Cloud, Azure

### Architecture

```
┌─────────────────┐
│   Claude Code   │
└────────┬────────┘
         │
         │ MCP Protocol
         │
    ┌────┴─────┐
    │          │
┌───▼───┐ ┌───▼────┐
│ Jira  │ │   DB   │
│Server │ │ Server │
└───┬───┘ └───┬────┘
    │         │
┌───▼──────┐  │
│ Jira API │  │
└──────────┘  │
          ┌───▼────┐
          │  DB    │
          └────────┘
```

---

## MCP Server Types

### 1. HTTP/HTTPS Servers (Recommended)

Most common and easiest to use.

**Example:**
```bash
claude mcp add --transport http sentry https://mcp.sentry.io/mcp
```

### 2. SSE (Server-Sent Events) Servers

Real-time updates.

**Example:**
```bash
claude mcp add --transport sse notifications https://notifications.company.com/sse
```

### 3. Local stdio Servers

Local processes communicating via stdin/stdout.

**Example:**
```bash
claude mcp add --transport stdio postgres npx @modelcontextprotocol/server-postgres
```

---

## Installing MCP Servers

### Installation Scopes

| Scope | Location | Shared | Use Case |
|-------|----------|--------|----------|
| `--local` | `.claude/mcp.json` | No | Project-specific, private |
| `--project` | `.claude/mcp.json` | Yes (git) | Team-shared |
| `--user` | `~/.claude/mcp.json` | No | Personal, all projects |

### Installation Examples

**Database Server:**
```bash
# PostgreSQL
claude mcp add \
  --transport stdio \
  --scope project \
  postgres \
  npx @modelcontextprotocol/server-postgres

# Then use:
> Query the users table for accounts created today
```

**Jira Integration:**
```bash
claude mcp add \
  --transport http \
  --scope user \
  jira \
  https://mcp.atlassian.com/jira

# Then use:
> Create a ticket for the bug we just found
```

**GitHub Integration:**
```bash
claude mcp add \
  --transport http \
  --scope project \
  github \
  https://mcp.github.com

# Then use:
> Create a pull request for these changes
```

---

## Banking IT Integrations

### Example 1: Internal Database Access

```bash
# Add PostgreSQL MCP server
claude mcp add \
  --transport stdio \
  --scope project \
  banking-db \
  npx @modelcontextprotocol/server-postgres \
  -- \
  --connection-string "postgresql://readonly@db.bank.internal:5432/banking"

# Configure environment
export DB_PASSWORD="${DB_READONLY_PASSWORD}"
```

**Usage:**
```
> Query the transactions table for suspicious activity in the last hour

> Get account balances for accounts with negative balance

> Show me the database schema for the payments table
```

### Example 2: Jira Integration

```bash
claude mcp add \
  --transport http \
  --scope user \
  jira \
  https://jira.bank.internal/mcp

# Set credentials
export JIRA_TOKEN="${MY_JIRA_TOKEN}"
```

**Usage:**
```
> Create a Jira ticket for the security vulnerability we just found

> Update JIRA-123 with deployment notes

> List all open tickets assigned to me
```

### Example 3: Internal API Access

Create custom MCP server for internal APIs:

`mcp-servers/bank-api-server.js`:
```javascript
#!/usr/bin/env node
const { MCPServer } = require('@modelcontextprotocol/sdk');

const server = new MCPServer({
  name: 'BankAPI',
  version: '1.0.0',
  tools: {
    getAccount: {
      description: 'Get account details',
      parameters: {
        accountId: { type: 'string' }
      },
      handler: async ({ accountId }) => {
        const response = await fetch(
          `https://api.bank.internal/accounts/${accountId}`,
          {
            headers: {
              'Authorization': `Bearer ${process.env.API_TOKEN}`
            }
          }
        );
        return response.json();
      }
    },
    searchTransactions: {
      description: 'Search transactions',
      parameters: {
        accountId: { type: 'string' },
        startDate: { type: 'string' },
        endDate: { type: 'string' }
      },
      handler: async (params) => {
        // Implementation
      }
    }
  }
});

server.listen();
```

**Install:**
```bash
claude mcp add \
  --transport stdio \
  --scope project \
  bank-api \
  node ./mcp-servers/bank-api-server.js
```

### Example 4: Monitoring Integration

```bash
# Datadog
claude mcp add \
  --transport http \
  --scope project \
  datadog \
  https://mcp.datadoghq.com

export DD_API_KEY="${DATADOG_API_KEY}"
```

**Usage:**
```
> Check the error rate for the payment service in the last 24 hours

> Show me the slowest API endpoints

> Are there any active alerts?
```

### Example 5: AWS Integration

```bash
claude mcp add \
  --transport http \
  --scope user \
  aws \
  https://mcp.aws.amazon.com

export AWS_PROFILE="bank-dev"
```

**Usage:**
```
> List all S3 buckets in us-east-1

> Check the status of the payment-processor Lambda function

> Show recent CloudWatch logs for the API Gateway
```

---

## Configuration File

MCP servers are stored in `.claude/mcp.json` (project) or `~/.claude/mcp.json` (user):

```json
{
  "servers": {
    "postgres": {
      "transport": "stdio",
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-postgres",
        "--connection-string",
        "postgresql://user@localhost:5432/banking"
      ],
      "env": {
        "PGPASSWORD": "${DB_PASSWORD}"
      }
    },
    "jira": {
      "transport": "http",
      "url": "https://jira.bank.internal/mcp",
      "auth": {
        "type": "bearer",
        "token": "${JIRA_TOKEN}"
      }
    }
  }
}
```

---

## Best Practices

### 1. Use Read-Only Access

```bash
# Good: Read-only database user
postgresql://readonly@db.bank.internal:5432/banking

# Bad: Admin access
postgresql://admin@db.bank.internal:5432/banking
```

### 2. Secure Credentials

```bash
# Good: Environment variables
export DB_PASSWORD="${VAULT_DB_PASSWORD}"

# Bad: Hardcoded in config
"password": "mypassword123"  # NEVER DO THIS
```

### 3. Project vs User Scope

```bash
# Project-wide (commit to git)
claude mcp add --scope project database ...

# Personal (not committed)
claude mcp add --scope user my-personal-jira ...
```

### 4. Test MCP Connections

```bash
# After adding MCP server
> List available MCP servers

> Test the database connection
```

### 5. Document MCP Servers

Create `.claude/MCP_SERVERS.md`:
```markdown
# MCP Servers

## PostgreSQL (banking-db)
- **Purpose**: Read-only access to production database
- **Credentials**: Set `DB_PASSWORD` environment variable
- **Usage**: Query database directly from Claude Code

## Jira (jira)
- **Purpose**: Create and update Jira tickets
- **Credentials**: Set `JIRA_TOKEN` environment variable
- **Usage**: Automate ticket creation

## Internal API (bank-api)
- **Purpose**: Access internal banking APIs
- **Credentials**: Set `API_TOKEN` environment variable
- **Usage**: Query account data, transactions
```

---

## Troubleshooting

### MCP Server Won't Connect

```bash
# Check MCP configuration
cat ~/.claude/mcp.json

# Test connection manually
curl https://mcp-server-url/health

# Check environment variables
echo $DB_PASSWORD
```

### Authentication Failures

```bash
# Verify credentials
export JIRA_TOKEN="your-token-here"

# Test authentication
curl -H "Authorization: Bearer $JIRA_TOKEN" https://jira.company.com/api/myself
```

### MCP Server Crashes

```bash
# Check logs
tail -f ~/.claude/mcp-logs/*.log

# Restart MCP server
claude mcp restart server-name
```

---

## Summary

In this section, you learned:

### Core Concepts
- MCP extends Claude Code with external integrations
- Three transport types: HTTP, SSE, stdio
- Three scopes: local, project, user

### Implementation
- Installing MCP servers
- Configuring authentication
- Environment variable usage

### Banking Applications
- Database integration (read-only)
- Jira ticket automation
- Internal API access
- Monitoring systems
- Cloud service integration

---

## Next Steps

1. **[Continue to Section 12: Version Control Integration](./12-git-integration.md)** - Git workflows
2. **[Review MCP Documentation](https://docs.claude.com/en/docs/claude-code/mcp)** - Official MCP guide
3. **Install your first MCP server** - Start with database or Jira

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-10

**Official References:**
- **MCP Guide**: https://docs.claude.com/en/docs/claude-code/mcp
- **MCP SDK**: https://github.com/anthropics/model-context-protocol
