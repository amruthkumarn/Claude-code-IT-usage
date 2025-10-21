# Section 10: Hooks & Automation

## Table of Contents
1. [Understanding Hooks](#understanding-hooks)
2. [Hook Types](#hook-types)
3. [Configuring Hooks](#configuring-hooks)
4. [Banking Automation Examples](#banking-automation-examples)
5. [Best Practices](#best-practices)

---

## Understanding Hooks

**Hooks** are scripts that run automatically at specific points in Claude Code's workflow, allowing you to:
- Validate changes before they happen
- Log actions for compliance
- Run automated tests
- Enforce policies
- Integrate with external systems

### Hook Lifecycle

```
User Request
     │
     ▼
┌────────────────┐
│  PreToolUse    │  ← Before Claude uses a tool
│  Hook          │    Can block or modify
└────────┬───────┘
         │
         ▼
┌────────────────┐
│  Tool Executes │
│  (Read, Edit,  │
│   Bash, etc.)  │
└────────┬───────┘
         │
         ▼
┌────────────────┐
│  PostToolUse   │  ← After tool completes
│  Hook          │    Can validate result
└────────┬───────┘
         │
         ▼
    Response
```

### Hook Capabilities

- **Block operations** - Prevent unsafe actions
- **Add context** - Inject additional information
- **Log events** - Audit trail for compliance
- **Validate** - Check code quality/security
- **Integrate** - Connect to external systems

---

## Hook Types

### 1. PreToolUse

Runs **before** a tool executes.

**Use cases:**
- Validate requests before execution
- Check permissions
- Run security scans
- Log attempted actions
- Block dangerous operations

### 2. PostToolUse

Runs **after** a tool completes.

**Use cases:**
- Validate results
- Run tests on changes
- Update documentation
- Send notifications
- Log completed actions

### 3. UserPromptSubmit

Runs when user submits a prompt.

**Use cases:**
- Pre-process user input
- Add context to requests
- Validate prompt content
- Track user interactions

---

## Configuring Hooks

### Basic Hook Configuration

`.claude/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate.sh",
            "description": "Validate code changes"
          }
        ]
      }
    ]
  }
}
```

### Hook Properties

```json
{
  "matcher": "Tool1|Tool2",  // Regex: which tools trigger this hook
  "hooks": [
    {
      "type": "command",           // Run a shell command
      "command": "./script.sh",    // Path to script
      "description": "What it does", // Optional: shown to user
      "blocking": true             // Optional: block on failure (default: true)
    }
  ]
}
```

### Hook Exit Codes

Your script's exit code controls Claude Code's behavior:

| Exit Code | Meaning | Claude Code Action |
|-----------|---------|-------------------|
| `0` | Success | Continue operation |
| `1-255` | Failure | Block operation, show error |

### Hook Output

**Simple (exit code only):**
```bash
#!/bin/bash
if grep -q "password.*=" $FILE; then
  echo "ERROR: Hardcoded password detected!"
  exit 1
fi
exit 0
```

**Advanced (JSON output):**
```bash
#!/bin/bash
cat <<EOF
{
  "block": true,
  "message": "Security violation: hardcoded credential",
  "context": {
    "file": "$FILE",
    "severity": "critical"
  }
}
EOF
exit 1
```

---

## Banking Automation Examples

### Example 1: Audit Logging

`.claude/scripts/audit-log.sh`:
```bash
#!/bin/bash
# Log all tool usage for compliance

TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
USER=$(whoami)
TOOL=${TOOL_NAME:-$1}
FILE=${FILE_PATH:-$2}
PROJECT=$(basename $(pwd))

LOG_FILE="$HOME/.claude/audit.log"
mkdir -p "$(dirname $LOG_FILE)"

# Log locally
echo "$TIMESTAMP | $USER | $PROJECT | $TOOL | $FILE" >> "$LOG_FILE"

# Send to central audit system (optional)
curl -X POST "https://audit.bank.internal/api/logs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $AUDIT_TOKEN" \
  -d "{
    \"timestamp\": \"$TIMESTAMP\",
    \"user\": \"$USER\",
    \"project\": \"$PROJECT\",
    \"tool\": \"$TOOL\",
    \"file\": \"$FILE\"
  }" 2>/dev/null || true

exit 0
```

**Configuration:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": ".*",
        "hooks": [{
          "type": "command",
          "command": "./.claude/scripts/audit-log.sh",
          "description": "Log for compliance audit"
        }]
      }
    ]
  }
}
```

### Example 2: Secrets Detection

`.claude/scripts/detect-secrets.sh`:
```bash
#!/bin/bash
# Prevent committing secrets

FILE=${FILE_PATH:-$1}

# Check for common secret patterns
if grep -qE '(password|secret|api_key|token)\s*=\s*["\047][^"\047]{8,}' "$FILE"; then
  echo "⚠️  Potential secret detected in $FILE"
  echo "Please use environment variables instead."
  exit 1
fi

# Check for AWS keys
if grep -qE 'AKIA[0-9A-Z]{16}' "$FILE"; then
  echo "⚠️  AWS Access Key detected in $FILE"
  exit 1
fi

# Check for private keys
if grep -q "BEGIN.*PRIVATE KEY" "$FILE"; then
  echo "⚠️  Private key detected in $FILE"
  exit 1
fi

exit 0
```

### Example 3: Code Quality Check

`.claude/scripts/quality-check.sh`:
```bash
#!/bin/bash
# Run linter and tests before changes

FILE=${FILE_PATH:-$1}

# Run ESLint
if [[ $FILE == *.js ]] || [[ $FILE == *.ts ]]; then
  npx eslint "$FILE" --max-warnings 0
  if [ $? -ne 0 ]; then
    echo "❌ Linting failed for $FILE"
    exit 1
  fi
fi

# Run Prettier
npx prettier --check "$FILE" > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "⚠️  File not formatted correctly. Run: npx prettier --write $FILE"
  exit 1
fi

echo "✅ Quality checks passed"
exit 0
```

**Configuration:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "./.claude/scripts/quality-check.sh",
          "description": "Run linter and formatter"
        }]
      }
    ]
  }
}
```

### Example 4: Test Automation

`.claude/scripts/run-tests.sh`:
```bash
#!/bin/bash
# Run tests after code changes

FILE=${FILE_PATH:-$1}

# Determine test file
TEST_FILE=$(echo "$FILE" | sed 's/\.ts$/.test.ts/' | sed 's/src\//tests\//')

if [ -f "$TEST_FILE" ]; then
  echo "Running tests for $FILE..."
  npm test -- "$TEST_FILE"

  if [ $? -ne 0 ]; then
    echo "❌ Tests failed for $FILE"
    exit 1
  fi

  echo "✅ All tests passed"
fi

exit 0
```

### Example 5: Database Change Validation

`.claude/scripts/validate-migration.sh`:
```bash
#!/bin/bash
# Validate database migrations

FILE=${FILE_PATH:-$1}

if [[ $FILE == *migration* ]]; then
  # Check for transaction wrappers
  if ! grep -q "BEGIN;" "$FILE"; then
    echo "❌ Migration missing BEGIN transaction"
    exit 1
  fi

  if ! grep -q "COMMIT;" "$FILE"; then
    echo "❌ Migration missing COMMIT"
    exit 1
  fi

  # Check for rollback script
  if ! grep -q "-- DOWN" "$FILE"; then
    echo "⚠️  Migration should include rollback (-- DOWN) section"
    exit 1
  fi

  echo "✅ Migration validation passed"
fi

exit 0
```

### Example 6: PCI-DSS Compliance Check

`.claude/scripts/pci-compliance.sh`:
```bash
#!/bin/bash
# Check for PCI-DSS violations

FILE=${FILE_PATH:-$1}

# Check for credit card numbers in logs
if grep -qE 'log.*\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b' "$FILE"; then
  echo "🚨 CRITICAL: Credit card number in logging detected!"
  echo "File: $FILE"
  echo "This violates PCI-DSS requirements."
  exit 1
fi

# Check for CVV storage
if grep -qiE '(cvv|cvc|card.*verification).*store|save' "$FILE"; then
  echo "🚨 CRITICAL: Potential CVV storage detected!"
  echo "CVV codes must NEVER be stored (PCI-DSS 3.2)."
  exit 1
fi

# Check for unencrypted cardholder data
if grep -qiE 'INSERT.*card_number|UPDATE.*card_number' "$FILE"; then
  if ! grep -q "encrypt" "$FILE"; then
    echo "⚠️  Warning: Card data should be encrypted"
    exit 1
  fi
fi

exit 0
```

**Configuration:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "./.claude/scripts/pci-compliance.sh",
          "description": "PCI-DSS compliance check"
        }]
      }
    ]
  }
}
```

---

## Best Practices

### 1. Keep Hooks Fast

```bash
# Good: Fast checks
if grep -q "pattern" $FILE; then
  exit 1
fi

# Bad: Slow operations
npm test  # Runs all tests (slow!)
```

### 2. Provide Clear Error Messages

```bash
# Good
echo "❌ Error: Hardcoded password on line 42"
echo "Use environment variables instead: PASSWORD=${DB_PASSWORD}"

# Bad
echo "Error"
```

### 3. Make Hooks Optional for Development

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{
          "command": "./scripts/strict-validation.sh",
          "blocking": false  // Warning only, doesn't block
        }]
      }
    ]
  }
}
```

### 4. Test Your Hooks

```bash
# Test hook script manually
FILE_PATH="src/test.ts" TOOL_NAME="Edit" ./.claude/scripts/my-hook.sh

# Should exit 0 (success) or 1 (failure) appropriately
echo "Exit code: $?"
```

### 5. Handle Errors Gracefully

```bash
#!/bin/bash
set -e  # Exit on error

# But handle expected failures
if ! command -v jq &> /dev/null; then
  echo "Warning: jq not installed, skipping JSON validation"
  exit 0  # Don't block if tool missing
fi

# Your validation logic here
```

### 6. Document Your Hooks

Create `.claude/hooks/README.md`:
```markdown
# Claude Code Hooks

## Audit Logging (audit-log.sh)
Logs all tool usage to ~/.claude/audit.log and central audit system.

## Secrets Detection (detect-secrets.sh)
Prevents committing credentials, API keys, and private keys.

## PCI Compliance (pci-compliance.sh)
Checks for credit card data in logs and CVV storage violations.

## Testing
Run hooks manually:
```bash
FILE_PATH="path/to/file" ./hooks/hook-name.sh
```
```

---

## Summary

In this section, you learned:

### Core Concepts
- Hooks automate validation and compliance
- Three hook types: PreToolUse, PostToolUse, UserPromptSubmit
- Hooks can block, log, or add context

### Implementation
- Configuring hooks in settings.json
- Writing hook scripts
- Exit codes and JSON output
- Environment variables available to hooks

### Banking Applications
- Audit logging for compliance
- Secrets detection
- PCI-DSS validation
- Code quality enforcement
- Test automation
- Database migration validation

---

## Next Steps

1. **[Continue to Section 11: MCP](../05-integration/11-mcp.md)** - External tool integration
2. **[Review Hooks Documentation](https://docs.claude.com/en/docs/claude-code/hooks)** - Official hooks guide
3. **Create your first hook** - Start with audit logging

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-9

**Official References:**
- **Hooks Guide**: https://docs.claude.com/en/docs/claude-code/hooks
