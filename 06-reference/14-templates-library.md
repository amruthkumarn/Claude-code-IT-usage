# Section 14: Templates Library

## Table of Contents
1. [Settings Templates](#settings-templates)
2. [Memory Templates](#memory-templates)
3. [Slash Command Templates](#slash-command-templates)
4. [Hook Script Templates](#hook-script-templates)
5. [Agent Configuration Templates](#agent-configuration-templates)

---

## Settings Templates

### Template 1: Standard Development

`.claude/settings.json`:
```json
{
  "$schema": "https://claude.ai/schemas/settings.json",
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write", "Bash"],
    "deny": ["WebSearch"]
  },
  "defaultModel": "sonnet",
  "env": {
    "NODE_ENV": "development"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "./.claude/hooks/lint-check.sh"
        }]
      }
    ]
  }
}
```

### Template 2: Production Read-Only

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch"]
  },
  "defaultModel": "opus",
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

### Template 3: Security Audit

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": "*"
  },
  "defaultModel": "opus",
  "outputStyle": "security-focused",
  "agents": {
    "security-auditor": {
      "description": "Security vulnerability scanner",
      "prompt": "You are a security expert. Identify vulnerabilities in code.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    }
  }
}
```

---

## Memory Templates

### Template 1: Basic Project Memory

`.claude/CLAUDE.md`:
```markdown
# {{PROJECT_NAME}}

## Overview
{{Brief description of the project}}

## Technology Stack
- Language: {{TypeScript/JavaScript/Python/etc}}
- Framework: {{Express/React/Django/etc}}
- Database: {{PostgreSQL/MySQL/MongoDB/etc}}
- Testing: {{Jest/Pytest/etc}}

## Coding Standards

### Code Style
- Use {{2/4}} spaces for indentation
- Maximum line length: {{80/100/120}}
- Follow {{Airbnb/Google/PEP8}} style guide

### TypeScript (if applicable)
- Use strict mode
- Prefer interfaces over types
- Explicit return types for public functions

## Security Requirements
- All endpoints require authentication
- Use parameterized queries for SQL
- Never log sensitive data (passwords, tokens, PII)
- Validate all user input

## Testing
- Minimum test coverage: {{70/80/90}}%
- Write tests before implementation (TDD)
- Mock external dependencies

## Git Workflow
- Branch naming: `feature/`, `fix/`, `refactor/`
- Commit format: {{Conventional Commits/Simple}}
- Require {{1/2}} PR approvals
```

### Template 2: Banking Project Memory

`.claude/CLAUDE.md`:
```markdown
# {{BANKING_PROJECT_NAME}}

## Compliance Requirements

### PCI-DSS
NEVER:
- Log credit card numbers (full or partial)
- Store CVV/CVV2 codes
- Store PIN blocks

ALWAYS:
- Encrypt cardholder data
- Use tokenization for storage
- Audit all access to payment data

### SOX
- All financial transactions must be auditable
- Change management required for production
- Separation of duties enforced

### GDPR
- Personal data encrypted at rest
- Right to deletion implemented
- Data retention: {{7}} years for financial records

## Security Standards
- Authentication: JWT with {{15}}-minute expiry
- Password requirements: {{12}}+ characters, mixed case, numbers, symbols
- Rate limiting: {{100}} requests/minute per IP
- Failed login lockout: {{5}} attempts

## Code Review Requirements
- Security review required for:
  - Authentication/authorization code
  - Payment processing
  - Database queries
  - External API integrations
- Automated security scan must pass
- No high/critical vulnerabilities

## Testing Standards
- Unit test coverage: ≥80%
- Integration tests for payment flows
- Never use real card numbers (use test cards: 4111111111111111)
- Mock external payment gateways
```

---

## Slash Command Templates

### Template 1: Security Review

`.claude/commands/security-review.md`:
```markdown
---
description: Comprehensive security review
---

Perform security analysis on: ${1:-.}

Check for:
## Authentication & Authorization
- Endpoints properly protected?
- JWT validation correct?
- Permissions checked?

## Input Validation
- SQL injection prevention?
- XSS prevention?
- CSRF protection?

## Data Protection
- Sensitive data encrypted?
- Secrets managed properly?
- Passwords hashed securely?

## Logging
- Security events logged?
- PII excluded from logs?

Provide file:line references and remediation steps.
```

### Template 2: Generate Tests

`.claude/commands/generate-tests.md`:
```markdown
---
description: Generate comprehensive tests - Usage: /generate-tests <file>
---

Generate unit tests for: $1

Include:
- Happy path tests
- Edge cases
- Error conditions
- Mock external dependencies

Use {{Jest/Pytest}} framework.
Follow AAA pattern (Arrange, Act, Assert).

Aim for {{80}}% coverage.
```

### Template 3: API Documentation

`.claude/commands/api-docs.md`:
```markdown
---
description: Generate OpenAPI documentation
---

Generate OpenAPI 3.0 documentation for: $1

Include:
- All endpoints (GET, POST, PUT, DELETE, PATCH)
- Request/response schemas with examples
- Authentication requirements
- Error responses (4xx, 5xx)
- Rate limiting information

Format as valid OpenAPI YAML.
```

---

## Hook Script Templates

### Template 1: Audit Logging

`.claude/hooks/audit-log.sh`:
```bash
#!/bin/bash
# Audit logging for compliance

TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
USER=$(whoami)
TOOL=${TOOL_NAME:-$1}
FILE=${FILE_PATH:-$2}

LOG_FILE="$HOME/.claude/audit.log"
mkdir -p "$(dirname $LOG_FILE)"

echo "$TIMESTAMP | $USER | $TOOL | $FILE" >> "$LOG_FILE"

# Optional: Send to central logging
# curl -X POST https://audit.company.com/api/logs ...

exit 0
```

### Template 2: Secrets Detection

`.claude/hooks/detect-secrets.sh`:
```bash
#!/bin/bash
# Detect hardcoded secrets

FILE=${FILE_PATH:-$1}

# Check for common secret patterns
if grep -qE '(password|secret|api_key|token)\s*=\s*["\047][^"\047]{8,}' "$FILE"; then
  echo "⚠️  Potential secret detected in $FILE"
  exit 1
fi

# Check for AWS keys
if grep -qE 'AKIA[0-9A-Z]{16}' "$FILE"; then
  echo "⚠️  AWS Access Key detected"
  exit 1
fi

exit 0
```

### Template 3: Lint Check

`.claude/hooks/lint-check.sh`:
```bash
#!/bin/bash
# Run linter before changes

FILE=${FILE_PATH:-$1}

if [[ $FILE == *.js ]] || [[ $FILE == *.ts ]]; then
  npx eslint "$FILE" --max-warnings 0
  if [ $? -ne 0 ]; then
    echo "❌ Linting failed"
    exit 1
  fi
fi

echo "✅ Lint check passed"
exit 0
```

---

## Agent Configuration Templates

### Template: Banking Agent Suite

`.claude/settings.json`:
```json
{
  "agents": {
    "compliance-checker": {
      "description": "PCI-DSS, SOX, GDPR compliance auditor",
      "prompt": "You are a banking compliance expert. Check for PCI-DSS, SOX, and GDPR compliance violations. Flag: CVV storage, unencrypted card data, missing audit trails, GDPR violations.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "sql-auditor": {
      "description": "Database security specialist",
      "prompt": "You are a database security expert. Identify: SQL injection vulnerabilities, missing parameterized queries, exposed sensitive data, missing indexes, N+1 queries.",
      "tools": ["Read", "Grep"],
      "model": "opus"
    },
    "security-reviewer": {
      "description": "General security auditor",
      "prompt": "You are a security expert. Identify: authentication bypasses, XSS, CSRF, insecure data handling, hardcoded secrets.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "test-generator": {
      "description": "Test creation specialist",
      "prompt": "You generate comprehensive tests with high coverage. Include happy paths, edge cases, error conditions. Use Jest/Pytest.",
      "tools": ["Read", "Write", "Grep"],
      "model": "sonnet"
    },
    "doc-generator": {
      "description": "Documentation specialist",
      "prompt": "You generate clear, comprehensive documentation. Include examples, API references, usage guides.",
      "tools": ["Read", "Write", "Grep"],
      "model": "sonnet"
    }
  }
}
```

---

## Quick Start Templates

### New Project Setup

```bash
#!/bin/bash
# setup-claude.sh - Initialize Claude Code for project

# Create directory structure
mkdir -p .claude/{commands,hooks,output-styles}

# Create settings.json
cat > .claude/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  },
  "defaultModel": "sonnet"
}
EOF

# Create CLAUDE.md
cat > .claude/CLAUDE.md << 'EOF'
# {{PROJECT_NAME}}

## Coding Standards
- Style guide: {{YOUR_STYLE}}
- Testing: {{YOUR_FRAMEWORK}}

## Security
- Authentication required
- Input validation mandatory
EOF

# Update .gitignore
echo ".claude/settings.local.json" >> .gitignore

echo "✅ Claude Code initialized!"
echo "Next steps:"
echo "1. Edit .claude/CLAUDE.md with your standards"
echo "2. Run: claude"
```

---

## Summary

This section provided ready-to-use templates for:

- Settings configurations (dev, production, audit)
- Memory files (basic and banking-specific)
- Slash commands (security, testing, docs)
- Hook scripts (audit, secrets, linting)
- Agent configurations (compliance, security, testing)
- Quick setup scripts

Copy and customize these templates for your projects!

---

## Next Steps

1. **[Continue to Section 15: Troubleshooting & FAQ](./15-troubleshooting-faq.md)** - Common issues and solutions
2. **Copy templates to your project** - Adapt as needed
3. **Share with your team** - Standardize across projects

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-13
