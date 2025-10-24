# Phase 1.3.2: Security & Permissions Model

**Learning Objectives:**
- Understand banking IT security requirements
- Master the permission system (allow, requireApproval, deny)
- Configure tools and permissions appropriately
- Implement secrets management
- Set up audit logging for compliance

**Time Commitment:** 45 minutes

**Prerequisites:** Phase 1.3.1 completed

---

## Table of Contents
1. [Banking IT Security Requirements](#banking-it-security-requirements)
2. [Permission System](#permission-system)
3. [Available Tools](#available-tools)
4. [Permission Configuration Levels](#permission-configuration-levels)
5. [Banking IT Permission Templates](#banking-it-permission-templates)
6. [Secrets Management](#secrets-management)
7. [Approval Workflow](#approval-workflow)
8. [Audit Logging](#audit-logging)
9. [Security Best Practices](#security-best-practices)

---

## Banking IT Security Requirements

Claude Code is designed with **security-first** principles, essential for banking IT:

**Key Security Features:**
1. ✅ **Read-only by default** - Requires explicit approval for changes
2. ✅ **Granular permissions** - Control what tools can be used
3. ✅ **Approval workflow** - Review before any modification
4. ✅ **Audit logging** - Track all actions (hooks)
5. ✅ **Secret detection** - Prevent credential exposure
6. ✅ **Network restrictions** - Control external access

**Why This Matters for Banking:**
- **PCI-DSS Compliance**: Ensure credit card data security
- **SOX Compliance**: Audit trail for financial operations
- **GDPR Compliance**: Personal data protection
- **Internal Security**: Prevent accidental data exposure

---

## Permission System

**Three Permission Levels:**

### 1. `allow` - Auto-approved, no prompt

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"]
  }
}
```

**Characteristics:**
- No approval prompt
- Executes immediately
- Good for: Read-only operations
- Safe for: Exploring code, searching

**Use when:** You trust the operation completely (e.g., reading files)

### 2. `requireApproval` - Prompt for each use

```json
{
  "permissions": {
    "requireApproval": ["Edit", "Write", "Bash"]
  }
}
```

**Characteristics:**
- Prompts before each execution
- You review and approve
- **Recommended for banking IT development**
- Default for most tools

**Use when:** Operations modify code or execute commands

### 3. `deny` - Never allowed

```json
{
  "permissions": {
    "deny": ["WebSearch", "WebFetch"]
  }
}
```

**Characteristics:**
- Completely blocked
- No prompt, no execution
- Highest priority
- Good for: Production environments
- Compliance: Prevent data exfiltration

**Use when:** You want to completely block certain operations (e.g., web access in production)

---

## Available Tools

| Tool | Purpose | Default Permission | Banking Recommendation | Risk Level |
|------|---------|-------------------|----------------------|------------|
| **Read** | Read files | allow | ✅ allow | Low |
| **Grep** | Search file contents | allow | ✅ allow | Low |
| **Glob** | Find files by pattern | allow | ✅ allow | Low |
| **Edit** | Modify existing files | requireApproval | ✅ requireApproval | Medium |
| **Write** | Create new files | requireApproval | ✅ requireApproval | Medium |
| **Bash** | Execute shell commands | requireApproval | ✅ requireApproval | High |
| **Task** | Launch sub-agents | requireApproval | ✅ allow (for automation) | Low |
| **TodoWrite** | Manage task list | allow | ✅ allow | Low |
| **WebSearch** | Search internet | requireApproval | ❌ deny (production) | Medium |
| **WebFetch** | Fetch URLs | requireApproval | ❌ deny (production) | Medium |
| **NotebookEdit** | Edit Jupyter notebooks | requireApproval | ✅ requireApproval | Medium |

**Risk Level Explanation:**
- **Low**: Read-only operations, no modifications
- **Medium**: Creates or modifies files
- **High**: Executes arbitrary commands

---

## Permission Configuration Levels

**Hierarchy (most specific wins):**

```
1. Enterprise Managed Policies  ← Highest priority (IT/DevOps controlled)
   ↓
2. Command-line flags             ← Session override (--permission-mode)
   ↓
3. Project settings (local)       ← Personal (.claude/settings.local.json)
   ↓
4. Project settings (shared)      ← Team (.claude/settings.json)
   ↓
5. User settings                  ← Personal (~/.claude/settings.json)
   ↓
6. Default permissions            ← Fallback
```

**Example Scenario:**
```
Enterprise policy says: "Deny WebFetch"
Your user settings say: "Allow WebFetch" → BLOCKED by enterprise
Project settings say: "Allow Read, Grep only"
CLI flag says: "--permission-mode plan" → CLI WINS for this session
```

---

## Banking IT Permission Templates

### Development Environment

**.claude/settings.json** (Team-shared):
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write", "Bash"],
    "deny": ["WebSearch", "WebFetch"]
  }
}
```

**Why:**
- Read operations allowed for speed
- Modifications require approval
- No external web access (data exfiltration prevention)

### Production (Read-Only)

**.claude/settings.json**:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebSearch", "WebFetch", "Task"]
  }
}
```

**Why:**
- Only reading/searching allowed
- No modifications possible
- No command execution
- No external access

### High-Security (Plan Mode)

**Command-line override:**
```bash
claude --permission-mode plan
```

**Equivalent to:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["*"]  // Everything else denied
  }
}
```

**Use for:**
- Analyzing production code
- Security audits
- Learning unfamiliar codebases

---

## Secrets Management

**Banking IT Policy: NEVER commit secrets**

### Bad ❌

```python
# config.py - WRONG!
API_KEY = "sk-1234567890"
DB_PASSWORD = "password123"
AWS_SECRET_KEY = "AKIAIOSFODNN7EXAMPLE"
```

**Problems:**
- Secrets exposed in version control
- Violates security policies
- PCI-DSS/SOX compliance issues

### Good ✅

```python
# config.py - CORRECT
import os

API_KEY = os.getenv("API_KEY")
DB_PASSWORD = os.getenv("DB_PASSWORD")
AWS_SECRET_KEY = os.getenv("AWS_SECRET_KEY")

# Validate at startup
if not all([API_KEY, DB_PASSWORD, AWS_SECRET_KEY]):
    raise ValueError("Missing required environment variables")
```

**Benefits:**
- Secrets stored securely outside code
- Different secrets per environment
- Complies with banking security policies

### Claude Code Configuration

**.claude/settings.json**:
```json
{
  "env": {
    "API_KEY": "${API_KEY}",
    "DB_PASSWORD": "${DB_PASSWORD}",
    "AWS_SECRET_KEY": "${AWS_SECRET_KEY}"
  }
}
```

**Set in your shell:**
```bash
export API_KEY="your-key-here"
export DB_PASSWORD="your-password-here"
export AWS_SECRET_KEY="your-secret-here"
```

---

## Approval Workflow

### Example: Claude wants to edit a file

```
> Add PII masking to the customer transformation

🔧 Tool Use: Edit
File: pipelines/transformations/customer.py
Changes:
  - Add mask_account_number() function
  - Apply masking in transform_customer()

[Diff shown here]

Approve this edit? (yes/no/always/never)
```

### Your options:

| Response | Meaning | Duration |
|----------|---------|----------|
| `yes` | Approve this one time | This operation only |
| `no` | Reject this change | This operation only |
| `always` | Always approve Edit tool | This session only |
| `never` | Never approve Edit tool | This session only |

**Best Practice:** Review diffs carefully before approving, especially for:
- Security-sensitive code
- Database operations
- Configuration changes
- Production pipelines

---

## Audit Logging

### For Banking Compliance (SOX)

Enable audit logging in **.claude/settings.json**:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": ".*",
      "hooks": [{
        "type": "command",
        "command": "./.claude/hooks/audit-log.sh"
      }]
    }]
  }
}
```

### Audit Log Script

**.claude/hooks/audit-log.sh**:
```bash
#!/bin/bash
# Log all Claude Code tool usage

LOGFILE=".claude/audit.log"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
TOOL="$1"
USER=$(whoami)

echo "$TIMESTAMP | $USER | $TOOL | $PWD" >> "$LOGFILE"
```

**Make executable:**
```bash
chmod +x .claude/hooks/audit-log.sh
```

### Audit Log Output

**.claude/audit.log**:
```
2025-10-23 14:30:15 | jdoe | Edit | /home/jdoe/payment-pipeline
2025-10-23 14:30:22 | jdoe | Bash | /home/jdoe/payment-pipeline
2025-10-23 14:31:05 | jdoe | Write | /home/jdoe/payment-pipeline
```

**Benefits:**
- SOX compliance (audit trail)
- Track all modifications
- Identify who changed what and when
- Support forensic analysis

---

## Security Best Practices

### 1. Least Privilege

```json
// Start restrictive, expand as needed
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  }
}
```

### 2. Secret Scanning

Set up hooks to detect secrets before commits:

**.claude/hooks/detect-secrets.sh**:
```bash
#!/bin/bash
if grep -r "sk-ant-\|AKIA\|password\s*=" pipelines/; then
  echo "❌ Possible secret detected!"
  exit 1
fi
```

### 3. Code Review

Always review diffs before approving edits:
- ✅ Check for hardcoded credentials
- ✅ Verify PII handling
- ✅ Confirm compliance with standards

### 4. Environment Separation

```bash
# Development
claude --permission-mode standard

# Production (read-only)
claude --permission-mode plan
```

### 5. Network Restrictions

```json
{
  "permissions": {
    "deny": ["WebSearch", "WebFetch"]  // Prevent data exfiltration
  }
}
```

**Why:** Prevents accidental or malicious data leakage to external services

---

## Role-Based Permission Examples

### Junior Developer

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash", "WebFetch"]
  }
}
```

### Senior Developer

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit", "Write", "Task"],
    "requireApproval": ["Bash"],
    "deny": ["WebFetch", "WebSearch"]
  }
}
```

### DevOps Engineer

```json
{
  "permissions": {
    "allow": [
      "Read", "Grep", "Glob",
      "Edit", "Write",
      "Bash",
      "Task"
    ],
    "requireApproval": ["Bash"]
  }
}
```

### Security Auditor (Read-Only)

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch", "NotebookEdit"]
  }
}
```

---

## Summary

In this subsection, you learned:

### Security Foundations
- ✅ Banking IT security requirements
- ✅ Permission system (allow, requireApproval, deny)
- ✅ Tool risk levels and recommendations

### Configuration
- ✅ Permission configuration hierarchy
- ✅ Banking IT permission templates
- ✅ Secrets management best practices

### Compliance
- ✅ Approval workflow
- ✅ Audit logging for SOX compliance
- ✅ Security best practices

### Key Takeaways
- **Default to least privilege** - Start restrictive
- **Never commit secrets** - Use environment variables
- **Enable audit logging** - Required for banking compliance
- **Review before approving** - Inspect all diffs
- **Separate environments** - Different permissions for dev/prod

---

## Next Steps

👉 **[Continue to 1.3.3: Memory & Context Management](./03-03-memory-context.md)**

**Quick Practice:**
1. Create `.claude/settings.json` with dev permissions
2. Set up audit logging hook
3. Test approval workflow by asking Claude to edit a file

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Detailed settings.json configuration
- [Phase 1.3.8: Hooks & Automation](./03-08-hooks-automation.md) - Advanced audit logging
- [Phase 1.2: Installation](./02-installation.md) - Initial setup

---

**Last Updated:** 2025-10-23
**Version:** 1.0
