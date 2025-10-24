# Phase 1.3.4: CLI Reference & Commands

**Learning Objectives:**
- Master Claude Code command-line syntax
- Learn essential commands and flags
- Understand permission modes
- Configure environment variables
- Use keyboard shortcuts effectively

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.3.1-1.3.3 completed

---

## Table of Contents
1. [Command-Line Syntax](#command-line-syntax)
2. [Essential Commands](#essential-commands)
3. [Permission Modes](#permission-modes)
4. [Model Selection](#model-selection)
5. [Working Directory](#working-directory)
6. [Configuration Files](#configuration-files)
7. [Environment Variables](#environment-variables)
8. [Keyboard Shortcuts](#keyboard-shortcuts)
9. [Built-In Slash Commands](#built-in-slash-commands)
10. [Exit Codes](#exit-codes)
11. [Common Command Patterns](#common-command-patterns)

---

## Command-Line Syntax

General syntax for Claude Code:
```bash
claude [OPTIONS] [INITIAL_PROMPT]
```

**Examples:**
```bash
# Start interactive session
claude

# Start with initial question
claude "What does this PySpark pipeline do?"

# Start in specific directory
cd ~/projects/payment-pipeline
claude

# Start in read-only mode
claude --plan

# Print mode (non-interactive)
claude -p "Generate PySpark schema for customer table"
```

---

## Essential Commands

### Start Interactive Session

```bash
# Default interactive mode
claude

# Start with initial question
claude "What does this PySpark pipeline do?"

# Start in specific directory
cd ~/projects/payment-pipeline
claude
```

### Print Mode (Non-Interactive)

```bash
# Single query, print result, exit
claude -p "Generate PySpark schema for customer table"

# Long form
claude --print "Explain this error: ..."

# Save output to file
claude -p "Generate payment validation function" > pipelines/validators/payment.py

# Use in scripts
TEST_CODE=$(claude -p "Generate pytest for transaction_processor.py")
echo "$TEST_CODE" > tests/test_transaction_processor.py
```

**Use cases:**
- Scripting and automation
- CI/CD pipelines
- Quick code generation
- Batch processing

### Version & Updates

```bash
# Check version
claude --version
claude -v

# Update to latest
claude update
```

### Help

```bash
# General help
claude --help
claude -h

# Help for specific topic
claude --help permissions
claude --help models
```

---

## Permission Modes

### Plan Mode (Read-Only)

```bash
# Explore code safely, no modifications
claude --permission-mode plan

# Alias
claude --plan

# Use case: Production code analysis
claude --plan
> Analyze the payment processing pipeline for bottlenecks
> What would you optimize?
```

**Characteristics:**
- ✅ Read-only access
- ✅ No file modifications
- ✅ Safe for production
- ✅ No approval prompts

### Standard Mode (Default)

```bash
# Read files (auto-approved)
# Modify files (requires approval)
claude

# Equivalent to:
claude --permission-mode standard
```

**Characteristics:**
- ✅ Read operations allowed
- ✅ Modifications require approval
- ✅ Default for development

### Custom Permissions

```bash
# Allow specific tools only
claude --allow Read,Grep,Glob

# Deny specific tools
claude --deny WebSearch,WebFetch

# Require approval for all tools
claude --require-approval "*"
```

**Examples:**
```bash
# Development with restrictions
claude --allow Read,Grep,Glob,Edit,Write --deny Bash

# Security audit mode
claude --allow Read,Grep --deny "*"

# Full permissions (use carefully)
claude --allow "*"
```

---

## Model Selection

### Current Model (Sonnet 4.5)

```bash
# Use Sonnet (default, recommended for banking IT)
claude --model sonnet

# Sonnet is currently the only model available in AWS Bedrock
```

**Why Sonnet:**
- Best balance of speed and capability
- Optimized for code generation
- Available in AWS Bedrock (banking IT approved)
- Suitable for all data engineering tasks

---

## Working Directory

```bash
# Start in current directory
claude

# Start in specific directory
claude --directory ~/projects/payment-pipeline

# Short form
claude -d ~/projects/payment-pipeline
```

**Example:**
```bash
# Analyze production pipeline
cd /opt/production/pipelines
claude --plan

# Work on development branch
cd ~/dev/feature-branch
claude
```

---

## Configuration Files

### Show Current Configuration

```bash
# Show current configuration
claude --show-config
```

**Output:**
```json
{
  "model": "sonnet",
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  },
  "workingDirectory": "/Users/jdoe/projects/payment-pipeline"
}
```

### Use Custom Config File

```bash
# Use custom config file
claude --config .claude/custom-settings.json

# Ignore project config, use user config only
claude --no-project-config
```

**Use cases:**
- Testing different configurations
- Environment-specific settings
- Personal overrides

---

## Environment Variables

### Key Environment Variables for Banking IT

#### Authentication

```bash
# API Key
export ANTHROPIC_API_KEY="sk-ant-your-key"

# Verify
echo $ANTHROPIC_API_KEY
```

#### PySpark Environment

```bash
# Python executable for Spark
export PYSPARK_PYTHON=python3
export PYSPARK_DRIVER_PYTHON=python3

# Spark home directory
export SPARK_HOME=/opt/spark

# Spark configuration
export SPARK_ENV=development
```

#### Proxy Configuration

```bash
# Corporate proxy
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080
export NO_PROXY=localhost,127.0.0.1,.bank.internal
```

#### SSL Certificates

```bash
# Corporate CA bundle
export REQUESTS_CA_BUNDLE=/etc/ssl/certs/corporate-ca.crt
export SSL_CERT_FILE=/etc/ssl/certs/corporate-ca.crt
export PIP_CERT=/etc/ssl/certs/corporate-ca.crt
```

#### Claude Code Specific

```bash
# Debug mode
export CLAUDE_DEBUG=1

# Log level
export CLAUDE_LOG_LEVEL=DEBUG

# Config directory (override default)
export CLAUDE_CONFIG_DIR=~/.config/claude
```

### Complete Environment Setup Script

**File:** `~/.claude_env.sh`
```bash
#!/bin/bash
# Claude Code environment setup for banking IT

# Authentication
export ANTHROPIC_API_KEY="sk-ant-your-key"

# PySpark
export PYSPARK_PYTHON=python3
export PYSPARK_DRIVER_PYTHON=python3
export SPARK_HOME=/opt/spark

# Corporate network
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080
export NO_PROXY=localhost,127.0.0.1,.bank.internal

# SSL certificates
export REQUESTS_CA_BUNDLE=/etc/ssl/certs/corporate-ca.crt
export SSL_CERT_FILE=/etc/ssl/certs/corporate-ca.crt

echo "Claude Code environment configured"
```

**Usage:**
```bash
# Source before using Claude Code
source ~/.claude_env.sh
claude
```

---

## Keyboard Shortcuts

### In the REPL

| Shortcut | Action |
|----------|--------|
| `Ctrl+D` | Exit Claude Code |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+L` | Clear screen |
| `↑` / `↓` | Navigate command history |
| `Tab` | Auto-complete (file paths) |
| `?` | Show keyboard shortcuts |

### Command History

```bash
# Navigate previous commands
> Generate a function
> [Press ↑] # Shows "Generate a function"

# Edit and re-run
> [Edit previous command]
> Generate a class
```

---

## Built-In Slash Commands

### In the REPL

```bash
> /help          # Show available commands
> /exit          # Exit Claude Code
> /clear         # Clear conversation history
> /reset         # Reset session (fresh start)
> /login         # Check login status / re-login
> /models        # List available models
> /permissions   # Show current permissions
> /config        # View configuration
> /memory        # Edit memory files
> /output-style  # Change output style
```

### Usage Examples

```bash
# Get help
> /help

# Change model
> /model sonnet

# Clear history
> /clear

# Code review
> /review
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Invalid arguments |
| `3` | Authentication failed |
| `4` | Permission denied |
| `5` | Network error |
| `130` | User interrupt (Ctrl+C) |

**Usage in scripts:**
```bash
#!/bin/bash
claude -p "Generate schema"
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed with exit code: $?"
fi
```

---

## Common Command Patterns

### Data Engineering Workflows

#### Analyze Pipeline Performance

```bash
claude --plan
> Analyze pipelines/payment_aggregation.py for performance issues
```

#### Generate New Pipeline

```bash
claude
> Generate PySpark pipeline to aggregate daily transactions by merchant
```

#### Fix Bug in Existing Code

```bash
claude
> Debug the null pointer error in pipelines/customer_transform.py
```

#### Run Tests and Fix Failures

```bash
claude
> Run pytest tests/test_payment_validator.py
> Fix any failing tests
```

#### Optimize Slow Query

```bash
claude
> Optimize the join in pipelines/transaction_join.py
> It's taking 2 hours for 500M records
```

### Scripting Examples

#### Batch Code Generation

```bash
#!/bin/bash
# Generate multiple pipeline files

PIPELINES=("ingestion" "transformation" "aggregation")

for pipeline in "${PIPELINES[@]}"; do
    claude -p "Generate PySpark ${pipeline} pipeline" > "pipelines/${pipeline}.py"
    echo "Generated ${pipeline}.py"
done
```

#### CI/CD Integration

```bash
#!/bin/bash
# CI/CD pipeline step: Generate tests

claude -p "Generate pytest for $CHANGED_FILE" > "tests/test_${CHANGED_FILE}"

if [ $? -eq 0 ]; then
    echo "Tests generated successfully"
    pytest "tests/test_${CHANGED_FILE}"
else
    echo "Test generation failed"
    exit 1
fi
```

---

## Quick Reference Card

### Essential Commands

```bash
claude                                    # Start interactive
claude --plan                             # Read-only mode
claude -p "query"                         # Print mode
claude --model sonnet                     # Specify model
claude --help                             # Get help
```

### Common Workflows

```bash
claude "Explain this pipeline"            # Quick question
claude --plan --directory ~/prod/         # Explore production
claude -p "Generate schema" > schema.py   # Generate code
```

### Environment Setup

```bash
export ANTHROPIC_API_KEY="sk-ant-..."    # Set API key
export PYSPARK_PYTHON=python3             # Configure PySpark
export HTTP_PROXY=http://proxy:8080      # Set proxy
```

### Permission Modes

```bash
claude --plan                             # Read-only
claude --allow Read,Grep,Glob             # Limited tools
claude --deny WebSearch,WebFetch          # Block specific tools
```

---

## Troubleshooting

### Command Not Found

```bash
# Check installation
which claude

# Reinstall if needed
pip install --upgrade claude-code

# Check PATH
echo $PATH
```

### Permission Denied

```bash
# Check permissions
claude --show-config

# Use plan mode temporarily
claude --plan
```

### Authentication Issues

```bash
# Check API key
echo $ANTHROPIC_API_KEY

# Re-login
claude --login
```

---

## Summary

In this subsection, you learned:

### Command-Line Basics
- ✅ Claude Code syntax and options
- ✅ Essential commands (interactive, print mode, version)
- ✅ Help system

### Modes and Permissions
- ✅ Plan mode (read-only)
- ✅ Standard mode (default)
- ✅ Custom permissions

### Configuration
- ✅ Working directory options
- ✅ Configuration files
- ✅ Environment variables

### Interactive Features
- ✅ Keyboard shortcuts
- ✅ Built-in slash commands
- ✅ Command history

### Practical Usage
- ✅ Common workflows for data engineering
- ✅ Scripting patterns
- ✅ CI/CD integration

---

## Next Steps

👉 **[Continue to 1.3.5: Project Configuration](./03-05-project-configuration.md)**

**Quick Practice:**
1. Try different permission modes: `claude --plan`, `claude`
2. Use print mode to generate code: `claude -p "Generate a function"`
3. Explore keyboard shortcuts in an interactive session

---

**Related Sections:**
- [Phase 1.3.2: Security & Permissions](./03-02-security-permissions.md) - Permission system details
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Settings files
- [Phase 1.2: Installation](./02-installation.md) - Initial setup

---

**Last Updated:** 2025-10-23
**Version:** 1.0
