# Phase 1.3.8: Hooks & Automation

**Learning Objectives:**
- Understand hooks and their lifecycle
- Learn hook types (PreToolUse, PostToolUse, UserPromptSubmit)
- Configure hooks for automation and compliance
- Implement banking IT automation examples
- Set up audit logging and security checks

**Time Commitment:** 60 minutes

**Prerequisites:** Phase 1.3.1-1.3.7 completed

---

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
            "command": "./scripts/validate.py",
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
      "command": "./script.py",    // Path to Python script
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

---

## Banking Automation Examples

### Example 1: Audit Logging

`.claude/scripts/audit_log.py`:
```python
#!/usr/bin/env python3
"""Log all tool usage for compliance"""
import os
import sys
from datetime import datetime, timezone
from pathlib import Path

def main():
    timestamp = datetime.now(timezone.utc).isoformat()
    user = os.environ.get('USER', os.getlogin())
    tool_name = os.environ.get('TOOL_NAME', sys.argv[1] if len(sys.argv) > 1 else 'unknown')
    file_path = os.environ.get('FILE_PATH', sys.argv[2] if len(sys.argv) > 2 else '')
    project = Path.cwd().name

    # Create log directory
    log_file = Path.home() / '.claude' / 'audit.log'
    log_file.parent.mkdir(parents=True, exist_ok=True)

    # Log locally
    log_entry = f"{timestamp} | {user} | {project} | {tool_name} | {file_path}\n"
    with open(log_file, 'a') as f:
        f.write(log_entry)

    sys.exit(0)

if __name__ == "__main__":
    main()
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
          "command": "python ./.claude/scripts/audit_log.py",
          "description": "Log for compliance audit"
        }]
      }
    ]
  }
}
```

### Example 2: Secrets Detection

`.claude/scripts/detect_secrets.py`:
```python
#!/usr/bin/env python3
"""Prevent committing secrets"""
import os
import sys
import re

def main():
    file_path = os.environ.get('FILE_PATH', sys.argv[1] if len(sys.argv) > 1 else '')

    if not os.path.exists(file_path):
        sys.exit(0)

    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        content = f.read()

    # Check for common secret patterns
    secret_pattern = re.compile(r'(password|secret|api_key|token)\s*=\s*["\'][^"\']{8,}["\']', re.IGNORECASE)
    if secret_pattern.search(content):
        print(f"WARNING: Potential secret detected in {file_path}")
        print("Please use environment variables instead.")
        sys.exit(1)

    # Check for AWS keys
    aws_pattern = re.compile(r'AKIA[0-9A-Z]{16}')
    if aws_pattern.search(content):
        print(f"WARNING: AWS Access Key detected in {file_path}")
        sys.exit(1)

    # Check for private keys
    if "BEGIN" in content and "PRIVATE KEY" in content:
        print(f"WARNING: Private key detected in {file_path}")
        sys.exit(1)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### Example 3: Code Quality Check

`.claude/scripts/quality_check.py`:
```python
#!/usr/bin/env python3
"""Run linter and formatter before changes"""
import os
import sys
import subprocess
from pathlib import Path

def run_command(cmd, description):
    """Run command and handle errors"""
    try:
        result = subprocess.run(
            cmd,
            shell=True,
            capture_output=True,
            text=True,
            timeout=60
        )
        if result.returncode != 0:
            print(f"FAILED: {description}")
            if result.stderr:
                print(result.stderr)
            return False
        return True
    except subprocess.TimeoutExpired:
        print(f"TIMEOUT: {description}")
        return False
    except Exception as e:
        print(f"ERROR: {description} - {e}")
        return False

def main():
    file_path = os.environ.get('FILE_PATH', sys.argv[1] if len(sys.argv) > 1 else '')

    if not file_path or not os.path.exists(file_path):
        sys.exit(0)

    file_ext = Path(file_path).suffix

    # Run Ruff linter for Python files
    if file_ext == '.py':
        if not run_command(f'ruff check {file_path}', 'Ruff linting'):
            print(f"FAILED: Linting failed for {file_path}")
            print(f"Fix with: ruff check --fix {file_path}")
            sys.exit(1)

        # Run Black formatter check
        if not run_command(f'black --check {file_path}', 'Black formatting'):
            print(f"WARNING: File not formatted correctly")
            print(f"Run: black {file_path}")
            sys.exit(1)

    print("SUCCESS: Quality checks passed")
    sys.exit(0)

if __name__ == "__main__":
    main()
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
          "command": "python ./.claude/scripts/quality_check.py",
          "description": "Run linter and formatter"
        }]
      }
    ]
  }
}
```

### Example 4: PCI-DSS Compliance Check

`.claude/scripts/pci_compliance.py`:
```python
#!/usr/bin/env python3
"""Check for PCI-DSS violations"""
import os
import sys
import re

def main():
    file_path = os.environ.get('FILE_PATH', sys.argv[1] if len(sys.argv) > 1 else '')

    if not file_path or not os.path.exists(file_path):
        sys.exit(0)

    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        content = f.read()

    # Check for credit card numbers in logs
    cc_pattern = re.compile(r'(log|print|logger).*\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b', re.IGNORECASE)
    if cc_pattern.search(content):
        print("CRITICAL: Credit card number in logging detected!")
        print(f"File: {file_path}")
        print("This violates PCI-DSS requirements.")
        sys.exit(1)

    # Check for CVV storage
    cvv_pattern = re.compile(r'(cvv|cvc|card.*verification).*(store|save|insert|update)', re.IGNORECASE)
    if cvv_pattern.search(content):
        print("CRITICAL: Potential CVV storage detected!")
        print("CVV codes must NEVER be stored (PCI-DSS 3.2).")
        sys.exit(1)

    # Check for PySpark DataFrame operations with sensitive data
    if 'pyspark' in content.lower():
        # Check for .show() on sensitive columns
        sensitive_show = re.compile(r'\.select\([^)]*(?:card|account|ssn)[^)]*\)\.show\(', re.IGNORECASE)
        if sensitive_show.search(content):
            print("WARNING: Displaying sensitive data with .show()")
            print("Use .show(truncate=True) or avoid showing sensitive columns.")

    sys.exit(0)

if __name__ == "__main__":
    main()
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
          "command": "python ./.claude/scripts/pci_compliance.py",
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

```python
# Good: Fast checks
import os
import sys

file_path = sys.argv[1]
with open(file_path, 'r') as f:
    if 'pattern' in f.read():
        sys.exit(1)

# Bad: Slow operations
import subprocess
subprocess.run(['pytest', '.'])  # Runs all tests (slow!)
```

### 2. Provide Clear Error Messages

```python
# Good
print("FAILED: Hardcoded password on line 42")
print("Use environment variables instead: PASSWORD=os.environ.get('DB_PASSWORD')")

# Bad
print("Error")
```

### 3. Make Hooks Optional for Development

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{
          "command": "python ./scripts/strict_validation.py",
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
FILE_PATH="pipelines/test.py" TOOL_NAME="Edit" python ./.claude/scripts/my_hook.py

# Should exit 0 (success) or 1 (failure) appropriately
echo "Exit code: $?"
```

### 5. Handle Errors Gracefully

```python
#!/usr/bin/env python3
import sys
import shutil

# Check for required tools
if not shutil.which('ruff'):
    print("Warning: ruff not installed, skipping linting")
    sys.exit(0)  # Don't block if tool missing

# Your validation logic here
```

### 6. Document Your Hooks

Create `.claude/hooks/README.md`:
```markdown
# Claude Code Hooks

## Audit Logging (audit_log.py)
Logs all tool usage to ~/.claude/audit.log and central audit system.

**Requirements:** `requests` library
```bash
pip install requests
```

## Secrets Detection (detect_secrets.py)
Prevents committing credentials, API keys, and private keys.

**Requirements:** None (uses stdlib only)

## PCI Compliance (pci_compliance.py)
Checks for credit card data in logs and CVV storage violations.

**Requirements:** None (uses stdlib only)

## Testing
Run hooks manually:
```bash
FILE_PATH="pipelines/transactions.py" python ./scripts/quality_check.py
```
```

---

## Summary

In this subsection, you learned:

### Core Concepts
- ✅ Hooks automate validation and compliance
- ✅ Three hook types: PreToolUse, PostToolUse, UserPromptSubmit
- ✅ Hooks can block, log, or add context

### Implementation
- ✅ Configuring hooks in settings.json
- ✅ Writing Python hook scripts
- ✅ Exit codes and error handling
- ✅ Environment variables available to hooks

### Banking Applications
- ✅ Audit logging for compliance
- ✅ Secrets detection
- ✅ PCI-DSS validation
- ✅ Code quality enforcement with Ruff and Black
- ✅ PySpark-specific validations

---

## Next Steps

👉 **[Continue to 1.3.9: MCP Integration](./03-09-mcp-integration.md)**

**Quick Practice:**
1. Create audit logging hook
2. Create secrets detection hook
3. Test hooks manually

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Hook configuration
- [Phase 1.3.9: MCP Integration](./03-09-mcp-integration.md) - External integrations
- [Phase 1.3.2: Security & Permissions](./03-02-security-permissions.md) - Security model

---

**Last Updated:** 2025-10-24
**Version:** 1.0
