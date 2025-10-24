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
claude mcp add --transport stdio postgres python -m mcp_server_postgres
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
# First install: pip install mcp-server-postgres
claude mcp add \
  --transport stdio \
  --scope project \
  postgres \
  python -m mcp_server_postgres

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
# First install: pip install mcp-server-postgres
claude mcp add \
  --transport stdio \
  --scope project \
  banking-db \
  python -m mcp_server_postgres \
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

`pipelines/mcp_servers/bank_api_server.py`:
```python
#!/usr/bin/env python3
"""
Banking API MCP Server
Provides secure access to internal banking APIs with PCI-DSS compliance.
"""
import os
import asyncio
from typing import Any, Dict
import httpx
from mcp.server import MCPServer
from mcp.types import Tool, TextContent


class BankAPIServer:
    """MCP server for internal banking API integration."""

    def __init__(self):
        self.server = MCPServer("BankAPI")
        self.api_token = os.environ.get("API_TOKEN")
        self.base_url = "https://api.bank.internal"

        # Register tools
        self._register_tools()

    def _register_tools(self):
        """Register available MCP tools."""

        @self.server.tool()
        async def get_account(account_id: str) -> Dict[str, Any]:
            """
            Get account details from banking API.

            Args:
                account_id: Account identifier (PCI-DSS Level 1 data)

            Returns:
                Account details dictionary
            """
            async with httpx.AsyncClient() as client:
                response = await client.get(
                    f"{self.base_url}/accounts/{account_id}",
                    headers={
                        "Authorization": f"Bearer {self.api_token}",
                        "X-Request-ID": f"mcp-{account_id}",
                        "X-Compliance": "PCI-DSS"
                    },
                    timeout=30.0
                )
                response.raise_for_status()
                return response.json()

        @self.server.tool()
        async def search_transactions(
            account_id: str,
            start_date: str,
            end_date: str
        ) -> Dict[str, Any]:
            """
            Search transactions for an account.

            Args:
                account_id: Account identifier
                start_date: Start date (ISO format: YYYY-MM-DD)
                end_date: End date (ISO format: YYYY-MM-DD)

            Returns:
                Transaction list with audit trail
            """
            async with httpx.AsyncClient() as client:
                response = await client.get(
                    f"{self.base_url}/transactions",
                    params={
                        "account_id": account_id,
                        "start_date": start_date,
                        "end_date": end_date
                    },
                    headers={
                        "Authorization": f"Bearer {self.api_token}",
                        "X-Compliance": "PCI-DSS,SOX",
                        "X-Audit": "enabled"
                    },
                    timeout=60.0
                )
                response.raise_for_status()
                return response.json()

    async def run(self):
        """Run the MCP server."""
        await self.server.run()


if __name__ == "__main__":
    server = BankAPIServer()
    asyncio.run(server.run())
```

**Install:**
```bash
# Install dependencies: pip install mcp httpx
claude mcp add \
  --transport stdio \
  --scope project \
  bank-api \
  python ./pipelines/mcp_servers/bank_api_server.py
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
      "command": "python",
      "args": [
        "-m",
        "mcp_server_postgres",
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
    },
    "bank-api": {
      "transport": "stdio",
      "command": "python",
      "args": [
        "./pipelines/mcp_servers/bank_api_server.py"
      ],
      "env": {
        "API_TOKEN": "${BANK_API_TOKEN}"
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


**Official References:**
- **MCP Guide**: https://docs.claude.com/en/docs/claude-code/mcp
- **MCP SDK**: https://github.com/anthropics/model-context-protocol
