# Section 4: CLI Reference

## Table of Contents
1. [Command Syntax](#command-syntax)
2. [Basic Commands](#basic-commands)
3. [Flags and Options](#flags-and-options)
4. [Interactive Mode Options](#interactive-mode-options)
5. [Permission Modes](#permission-modes)
6. [Model Selection](#model-selection)
7. [Working Directory Options](#working-directory-options)
8. [Agent Configuration](#agent-configuration)
9. [Environment Variables](#environment-variables)
10. [Exit Codes](#exit-codes)
11. [Complete Flag Reference](#complete-flag-reference)

---

## Command Syntax

The general syntax for Claude Code commands:

```bash
claude [OPTIONS] [INITIAL_PROMPT]
```

**Examples:**
```bash
# Interactive mode (default)
claude

# Start with an initial prompt
claude "What does this project do?"

# Print mode (SDK, non-interactive)
claude -p "Generate a regex for email validation"

# With options
claude --model sonnet --permission-mode plan
```

---

## Basic Commands

### Starting Interactive REPL

```bash
# Start in current directory
claude

# Start with initial query
claude "Explain the authentication system"
```

**What happens:**
- Opens interactive session
- Loads project context
- Shows welcome message
- Waits for your input

### Print Mode (SDK)

```bash
# Single query, print result, exit
claude -p "What is the purpose of this codebase?"

# Short form
claude --print "Generate unit tests for utils.js"
```

**Characteristics:**
- Non-interactive
- Single query only
- Prints response to stdout
- Exits immediately
- Useful for scripting

**Examples:**
```bash
# Save output to file
claude -p "Generate an API client for this OpenAPI spec" > api-client.ts

# Pipe input
cat error.log | claude -p "Explain these errors"

# Use in scripts
VALIDATION_CODE=$(claude -p "Generate email validation function")
echo "$VALIDATION_CODE" > src/utils/validation.js
```

### Version Information

```bash
# Show version
claude --version

# Or
claude -v
```

### Update Claude Code

```bash
# Update to latest version
claude update
```

---

## Flags and Options

### Help

```bash
# Show help
claude --help
claude -h

# Help for specific topic
claude --help permissions
```

### Verbose Output

```bash
# Show detailed information (debugging)
claude --verbose

# Show even more detail
claude --verbose --verbose
```

### Configuration

```bash
# Show current configuration
claude --show-config

# Use specific config file
claude --config /path/to/custom-config.json
```

---

## Interactive Mode Options

### Continuing Previous Session

```bash
# Continue most recent conversation
claude --continue

# Continue specific session by ID
claude --continue abc123
```

**Use cases:**
- Resume interrupted work
- Continue after reviewing changes
- Add to previous implementation

**Example workflow:**
```bash
# Session 1: Initial work
claude
> Add user authentication
> /exit

# Session 2: Continue later
claude --continue
> Now add password reset functionality
```

### Starting Fresh

```bash
# Start with clean history (default)
claude

# Explicitly start new session
claude --new
```

---

## Permission Modes

Control what Claude can do with the `--permission-mode` flag.

### Available Modes

| Mode | Read | Write | Execute | Use Case |
|------|------|-------|---------|----------|
| `interactive` | ✓ | Ask | Ask | Default - ask for approval |
| `auto-approve` | ✓ | ✓ | ✓ | Auto-approve all (dangerous!) |
| `plan` | ✓ | ✗ | ✗ | Read-only exploration |
| `deny` | ✓ | ✗ | ✗ | Read-only, reject all writes |

### Interactive Mode (Default)

```bash
claude --permission-mode interactive

# Or simply:
claude
```

**Behavior:**
- Allows reading without approval
- Asks before writing files
- Asks before executing commands
- Asks before network requests

**Best for:** Normal development work

### Auto-Approve Mode

```bash
claude --permission-mode auto-approve
```

**⚠️ Warning:** Claude can make any changes without asking!

**Behavior:**
- Automatically approves all actions
- No approval prompts
- Faster workflow
- Higher risk

**Best for:**
- Trusted, isolated environments
- Automated workflows
- Disposable test environments

**⚠️ Banking IT:** Only use in development containers or sandboxes!

### Plan Mode (Read-Only)

```bash
claude --permission-mode plan
```

**Behavior:**
- Can read files
- Can search and analyze
- **Cannot** write files
- **Cannot** execute commands
- **Cannot** make any changes

**Best for:**
- Exploring unfamiliar code
- Code reviews
- Understanding systems
- Auditing
- Learning

**Example:**
```bash
# Safely explore production codebase
cd ~/production/payment-processor
claude --permission-mode plan

> Analyze security of the authentication system
> Find all database queries
> Check for SQL injection vulnerabilities
```

### Deny Mode

```bash
claude --permission-mode deny
```

**Behavior:**
- Similar to plan mode
- Explicitly rejects write operations
- More strict than plan mode

---

## Model Selection

Choose which AI model to use with the `--model` flag.

### Available Models

**Current AWS Bedrock Availability:**

```bash
# Use Sonnet (default, currently available)
claude --model sonnet

# Default behavior (uses Sonnet)
claude
```

**🏦 Banking IT Note:** Only Sonnet is currently available in AWS Bedrock environments. Opus is in development, and Haiku is not currently available.

### Model Selection

**Sonnet (Currently Available):**
```bash
claude --model sonnet

# Suitable for:
# - All development tasks
# - Code reviews and refactoring
# - Bug fixes and debugging
# - Feature development
# - Security analysis
# - Performance optimization
# - Architectural decisions
```

### Future Model Availability

As additional models become available in AWS Bedrock, they will be documented here with specific use cases and selection strategies.

---

## Working Directory Options

Control which directories Claude can access.

### Default Behavior

```bash
# Current directory only
cd ~/projects/my-app
claude

# Claude can access: ~/projects/my-app/**
# Claude cannot access: ~/projects/other-app
```

### Adding Directories

```bash
# Add additional directories
claude --add-dir ../shared-lib --add-dir ../config

# Now Claude can access:
# - ~/projects/my-app/** (current)
# - ~/projects/shared-lib/**
# - ~/projects/config/**
```

### Multiple Projects

```bash
# Work across multiple related projects
cd ~/projects/frontend
claude \
  --add-dir ../backend \
  --add-dir ../shared-types \
  --add-dir ../config
```

**Example use case:**
```
> Update the API client in frontend to match the new backend endpoints
```

Claude can now read from both frontend and backend directories.

### Banking Example: Microservices

```bash
# Working on payment service that depends on shared libraries
cd ~/banking/payment-service
claude \
  --add-dir ../shared/auth \
  --add-dir ../shared/logging \
  --add-dir ../shared/types \
  --add-dir ../config/development

> Implement payment processing using the shared auth module
```

---

## Agent Configuration

Define custom sub-agents for specialized tasks using the `--agents` flag.

### Syntax

```bash
claude --agents '{"agent-name": {"description": "...", "prompt": "...", "tools": [...], "model": "..."}}'
```

### Basic Agent Definition

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer",
    "prompt": "You are a senior code reviewer. Focus on security, performance, and best practices."
  }
}'
```

Now you can use:
```
> @code-reviewer review the authentication code
```

### Agent with Tool Restrictions

```bash
claude --agents '{
  "security-auditor": {
    "description": "Security-focused auditor",
    "prompt": "You are a security expert. Analyze code for vulnerabilities.",
    "tools": ["Read", "Grep", "Glob"]
  }
}'
```

The `security-auditor` agent can only read files, not modify them.

### Agent with Specific Model

```bash
claude --agents '{
  "architect": {
    "description": "System architecture expert",
    "prompt": "You are a senior architect. Analyze system design and scalability.",
    "model": "sonnet"
  }
}'
```

The `architect` agent uses Claude Sonnet for architectural analysis.

### Multiple Agents

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Code quality expert",
    "prompt": "Review code for quality, maintainability, and best practices.",
    "tools": ["Read", "Grep"]
  },
  "test-writer": {
    "description": "Test generation specialist",
    "prompt": "Generate comprehensive unit and integration tests.",
    "model": "sonnet"
  },
  "security-auditor": {
    "description": "Security vulnerability scanner",
    "prompt": "Identify security vulnerabilities and compliance issues.",
    "tools": ["Read", "Grep", "Glob"],
    "model": "sonnet"
  }
}'
```

### Banking Example Agents

```bash
claude --agents '{
  "compliance-checker": {
    "description": "Banking compliance expert",
    "prompt": "You are a banking compliance expert. Check code for SOX, PCI-DSS, and regulatory compliance.",
    "tools": ["Read", "Grep", "Glob"],
    "model": "sonnet"
  },
  "sql-auditor": {
    "description": "Database security auditor",
    "prompt": "Audit SQL queries for injection vulnerabilities, performance issues, and data access patterns.",
    "tools": ["Read", "Grep"]
  }
}'
```

**Usage:**
```
> @compliance-checker review payment processing for PCI-DSS compliance
> @sql-auditor check all database queries for SQL injection
```

### Agent Configuration File

For complex agent setups, use a configuration file:

```bash
# Create agents.json
cat > agents.json << 'EOF'
{
  "code-reviewer": {
    "description": "Senior code reviewer",
    "prompt": "Review code for quality, security, and maintainability.",
    "tools": ["Read", "Grep", "Glob"]
  },
  "test-generator": {
    "description": "Test specialist",
    "prompt": "Generate comprehensive tests with high coverage.",
    "model": "sonnet"
  }
}
EOF

# Use the file
claude --agents "$(cat agents.json)"
```

---

## Environment Variables

Configure Claude Code via environment variables.

### Authentication

```bash
# API key authentication
export ANTHROPIC_API_KEY="sk-ant-..."

# Use in session
claude
```

### Proxy Configuration

```bash
# HTTP/HTTPS proxy
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1,.internal.company.com"

# Use proxy
claude
```

### SSL Certificates

```bash
# Custom CA certificate (corporate SSL inspection)
export NODE_EXTRA_CA_CERTS="/path/to/corporate-ca-bundle.crt"

claude
```

### Configuration Directory

```bash
# Custom config location
export CLAUDE_CONFIG_DIR="$HOME/.config/claude"

claude
```

### Debug Mode

```bash
# Enable debug output
export CLAUDE_DEBUG=1

claude
```

### Banking IT Environment Setup

**PowerShell Profile ($PROFILE):**
```powershell
# Corporate proxy
Set-Item -Path Env:HTTP_PROXY -Value "http://proxy.bank.com:8080"
Set-Item -Path Env:HTTPS_PROXY -Value "http://proxy.bank.com:8080"
Set-Item -Path Env:NO_PROXY -Value "localhost,127.0.0.1,.bank.internal"

# Corporate SSL certificate
Set-Item -Path Env:NODE_EXTRA_CA_CERTS -Value "C:\certs\bank-ca-bundle.crt"

# Claude API key (or use /login for interactive auth)
Set-Item -Path Env:ANTHROPIC_API_KEY -Value "your-api-key-here"
```

**WSL2 Profile (~/.bashrc):**
```bash
# Corporate proxy
export HTTP_PROXY="http://proxy.bank.com:8080"
export HTTPS_PROXY="http://proxy.bank.com:8080"
export NO_PROXY="localhost,127.0.0.1,.bank.internal"

# Corporate SSL certificate
export NODE_EXTRA_CA_CERTS="/etc/ssl/certs/bank-ca-bundle.crt"

# Claude API key (or use /login for interactive auth)
export ANTHROPIC_API_KEY="your-api-key-here"

# Optional: Enable audit logging
export CLAUDE_AUDIT_LOG="$HOME/.claude/audit.log"
```

---

## Exit Codes

Claude Code returns standard exit codes for scripting.

| Exit Code | Meaning |
|-----------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Invalid arguments |
| `3` | Authentication error |
| `4` | Permission denied |
| `5` | Network error |
| `130` | Interrupted (Ctrl+C) |

### Using in Scripts

```bash
#!/bin/bash

# Run Claude Code
claude -p "Generate validation function" > output.js

# Check exit code
if [ $? -eq 0 ]; then
  echo "Success!"
  mv output.js src/utils/validation.js
else
  echo "Failed with exit code $?"
  exit 1
fi
```

### Error Handling

```bash
#!/bin/bash

set -e  # Exit on error

# This will exit if Claude Code fails
claude -p "Analyze code" > analysis.txt

# Continue if successful
echo "Analysis complete"
```

---

## Complete Flag Reference

Comprehensive reference of all flags and options.

### Global Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--help` | `-h` | - | - | Show help message |
| `--version` | `-v` | - | - | Show version |
| `--verbose` | - | - | - | Verbose output (repeat for more detail) |
| `--config` | - | string | `~/.claude/settings.json` | Config file path |
| `--show-config` | - | - | - | Display current configuration |

### Mode Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--print` | `-p` | - | - | Print mode (non-interactive) |
| `--continue` | - | string (optional) | - | Continue previous session |
| `--new` | - | - | true | Start new session |

### Permission Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--permission-mode` | string | `interactive` | Permission mode: `interactive`, `auto-approve`, `plan`, `deny` |

### Model Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--model` | string | `sonnet` | AI model (currently only `sonnet` available in AWS Bedrock) |

### Directory Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--add-dir` | string[] | - | Add additional working directories (can be repeated) |

### Agent Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--agents` | JSON string | - | Define custom sub-agents |

### Advanced Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--append-system-prompt` | string | - | Append to system prompt |
| `--max-tokens` | number | - | Override max response tokens |
| `--timeout` | number | 120000 | Request timeout (ms) |
| `--no-color` | - | - | Disable colored output |
| `--json` | - | - | Output in JSON format (print mode) |

---

## Command Combinations

### Common Combinations

**Explore new codebase safely:**
```bash
claude --permission-mode plan --model sonnet
```

**Quick query with output:**
```bash
claude -p "Generate regex for phone validation" > phone-validator.js
```

**Continue previous session:**
```bash
claude --continue
```

**Multi-directory development:**
```bash
claude --add-dir ../shared --add-dir ../config
```

**Custom agents with restrictions:**
```bash
claude --agents '{"auditor": {"description": "Security auditor", "tools": ["Read", "Grep"]}}' --permission-mode plan
```

**Quick query with output:**
```bash
claude -p "Explain async/await" > async-explanation.md
```

### Banking IT Examples

**Secure code review:**
```bash
cd ~/banking/payment-service
claude \
  --permission-mode plan \
  --agents '{
    "compliance": {
      "description": "Compliance checker",
      "prompt": "Check for regulatory compliance",
      "tools": ["Read", "Grep"]
    }
  }'
```

**Development with shared libraries:**
```bash
cd ~/banking/frontend
claude \
  --add-dir ../backend-api \
  --add-dir ../shared-types \
  --add-dir ../auth-lib
```

**Automated security scan:**
```bash
cd ~/banking/transaction-processor
claude \
  --permission-mode plan \
  -p "Scan for SQL injection vulnerabilities, hardcoded secrets, and authentication bypass" \
  > security-report.txt
```

---

## Scripting with Claude Code

### Bash Script Example

```bash
#!/bin/bash
# generate-docs.sh - Auto-generate documentation

set -e

PROJECT_DIR="$1"

if [ -z "$PROJECT_DIR" ]; then
  echo "Usage: $0 <project-directory>"
  exit 1
fi

cd "$PROJECT_DIR"

echo "Generating documentation for $PROJECT_DIR..."

# Generate API documentation
claude -p "Generate API documentation for all endpoints" > docs/API.md

# Generate README
claude -p "Generate a comprehensive README based on the codebase" > README.md

# Generate architecture doc
claude -p "Create an architecture diagram and explanation" > docs/ARCHITECTURE.md

echo "Documentation generated successfully!"
```

### Python Script Example

```python
#!/usr/bin/env python3
# code-quality-check.py

import subprocess
import sys
import json

def run_claude(prompt):
    """Run Claude Code and return output"""
    result = subprocess.run(
        ['claude', '-p', prompt],
        capture_output=True,
        text=True,
        check=True
    )
    return result.stdout

def main():
    print("Running code quality checks...")

    # Check for security issues
    print("1. Security scan...")
    security = run_claude(
        "Scan for security vulnerabilities. "
        "Return JSON with findings."
    )

    # Check code coverage
    print("2. Test coverage analysis...")
    coverage = run_claude(
        "Analyze test coverage. "
        "Identify untested code."
    )

    # Check for code smells
    print("3. Code smell detection...")
    smells = run_claude(
        "Identify code smells and suggest refactoring."
    )

    # Generate report
    with open('quality-report.txt', 'w') as f:
        f.write("# Code Quality Report\n\n")
        f.write("## Security\n" + security + "\n\n")
        f.write("## Coverage\n" + coverage + "\n\n")
        f.write("## Code Smells\n" + smells + "\n")

    print("Report generated: quality-report.txt")

if __name__ == '__main__':
    main()
```

### CI/CD Integration

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Authenticate
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: echo "API key configured"

      - name: Review Changes
        run: |
          claude --permission-mode plan -p \
            "Review the git diff for: security issues, bugs, code quality problems. \
             Provide a structured report." \
            > review-report.txt

      - name: Post Review
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const report = fs.readFileSync('review-report.txt', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '## Claude Code Review\n\n' + report
            });
```

---

## Summary

In this section, you learned:

### Command Basics
- Interactive vs Print mode
- Starting and continuing sessions
- Help and version commands

### Configuration Options
- Permission modes (interactive, auto-approve, plan, deny)
- Model selection (Sonnet currently available in AWS Bedrock)
- Working directory management
- Custom agent configuration

### Advanced Usage
- Environment variables for configuration
- Exit codes for scripting
- Complete flag reference
- Common command combinations

### Practical Applications
- Banking IT specific configurations
- Scripting examples
- CI/CD integration
- Automated workflows

---

## Next Steps

1. **[Continue to Section 5: Project Configuration](../03-advanced/05-project-configuration.md)** - Configure `.claude/` directory
2. **[Jump to Section 6: Memory Management](../03-advanced/06-memory-management.md)** - Set up CLAUDE.md files
3. **[Review CLI Reference Docs](https://docs.claude.com/en/docs/claude-code/cli-reference)** - Official CLI documentation

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-3

**Official References:**
- **CLI Reference**: https://docs.claude.com/en/docs/claude-code/cli-reference
- **Interactive Mode**: https://docs.claude.com/en/docs/claude-code/interactive-mode
- **Settings Documentation**: https://docs.claude.com/en/docs/claude-code/settings
