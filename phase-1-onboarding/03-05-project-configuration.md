# Phase 1.3.5: Project Configuration

**Learning Objectives:**
- Understand the .claude directory structure
- Master settings hierarchy and precedence
- Configure team-shared and personal settings
- Set up permission profiles for banking IT
- Customize output styles

**Time Commitment:** 60 minutes

**Prerequisites:** Phase 1.3.1-1.3.4 completed

---

## ⚡ Quick Start (5 minutes)

**Goal:** Set up a complete .claude directory in 5 minutes.

### Try This Right Now

```bash
# 1. Create a new project
mkdir -p ~/my-banking-project
cd ~/my-banking-project

# 2. One-command setup
mkdir -p .claude/{commands,scripts}
cat > .claude/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  }
}
EOF

# 3. Test it
claude
> /config
# You'll see your new configuration!
```

**What just happened?**
- Created `.claude/` directory structure
- Set up team-shared configuration
- Configured safe permissions (read allowed, changes require approval)

**Next:** Let's understand the complete configuration system...

---

## Table of Contents
1. [The .claude Directory](#the-claude-directory)
2. [Settings Hierarchy](#settings-hierarchy)
3. [settings.json (Team-Shared)](#settingsjson-team-shared)
4. [settings.local.json (Personal)](#settingslocaljson-personal)
5. [Permission Configuration](#permission-configuration)
6. [Environment Variables](#environment-variables)
7. [Output Styles](#output-styles)
8. [Banking IT Templates](#banking-it-templates)
9. [Best Practices](#best-practices)

---

## The .claude Directory

The `.claude/` directory in your project root contains project-specific configuration for Claude Code.

### Directory Structure

```
your-project/
├── .claude/
│   ├── settings.json           # Team-shared settings (commit to git)
│   ├── settings.local.json     # Personal settings (add to .gitignore)
│   ├── CLAUDE.md               # Project memory (commit to git)
│   ├── commands/               # Custom slash commands
│   │   ├── review.md
│   │   ├── test.md
│   │   └── deploy.md
│   └── output-styles/          # Custom output styles
│       └── detailed.md
├── .gitignore
└── ...
```

### Creating the .claude Directory

```bash
# Navigate to project root
cd ~/projects/banking-etl

# Create .claude directory structure
mkdir -p .claude/commands .claude/output-styles

# Create settings files
touch .claude/settings.json
touch .claude/settings.local.json
touch .claude/CLAUDE.md

# Add to .gitignore
echo ".claude/settings.local.json" >> .gitignore
```

---

## 🔨 Exercise 1: Complete .claude Directory Setup (20 minutes)

**Goal:** Build a production-ready .claude configuration for a banking project.

### Step 1: Create comprehensive directory structure

```bash
cd ~/banking-etl-project
mkdir -p .claude/{commands,scripts,output-styles}
mkdir -p {pipelines,tests,schemas,config}
```

### Step 2: Create settings.json (Team Configuration)

```bash
cat > .claude/settings.json << 'EOF'
{
  "$schema": "https://claude.ai/schemas/settings.json",
  "_comment": "Banking ETL Project - Team Configuration",
  "_lastUpdated": "2025-10-24",

  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write", "Bash"],
    "deny": ["WebSearch", "WebFetch"]
  },

  "defaultModel": "sonnet",

  "env": {
    "ENVIRONMENT": "development",
    "LOG_LEVEL": "debug",
    "SPARK_HOME": "/opt/spark",
    "PYSPARK_PYTHON": "python3"
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/scripts/pre-edit-check.sh",
            "description": "Run linter before file changes"
          }
        ]
      }
    ]
  },

  "outputStyle": "default"
}
EOF
```

### Step 3: Create settings.local.json (Personal Overrides)

```bash
cat > .claude/settings.local.json << 'EOF'
{
  "_comment": "Personal settings - NOT committed to git",

  "env": {
    "MY_DEV_API_KEY": "dev-key-placeholder"
  },

  "outputStyle": "detailed"
}
EOF
```

### Step 4: Create CLAUDE.md

```bash
cat > .claude/CLAUDE.md << 'EOF'
# Banking ETL Project

## Tech Stack
- Python 3.10+
- PySpark 3.5+
- Delta Lake

## Coding Standards
- Use type hints
- Follow PEP 8
- Use Decimal for money amounts
- Mask PII in logs

## Security
- No hardcoded secrets
- All transactions must have audit trail
EOF
```

### Step 5: Create a pre-edit check script

```bash
cat > .claude/scripts/pre-edit-check.sh << 'EOF'
#!/bin/bash
# Pre-edit validation script

echo "Running pre-edit checks..."

# Check if ruff is available
if command -v ruff &> /dev/null; then
    echo "✓ Linter available"
else
    echo "⚠ Warning: ruff not installed"
fi

# Check for common issues
if grep -r "password\s*=" . 2>/dev/null | grep -v ".git" | grep -v "scripts" >/dev/null; then
    echo "❌ Warning: Possible hardcoded password detected"
fi

exit 0
EOF

chmod +x .claude/scripts/pre-edit-check.sh
```

### Step 6: Configure .gitignore

```bash
cat >> .gitignore << 'EOF'

# Claude Code
.claude/settings.local.json
.claude/.sessions/
.claude/logs/
EOF
```

### Step 7: Test your configuration

```bash
claude

> /config
```

**You should see:**
- Your permissions configuration
- Environment variables
- Hook configuration
- Default model setting

### Step 8: Test the hooks

```
> Create a file called test.py with a simple function
```

**Expected:** You'll see "Running pre-edit checks..." from your hook!

### ✅ Checkpoint
- [ ] Created complete .claude/ directory structure
- [ ] Configured team settings (settings.json)
- [ ] Configured personal settings (settings.local.json)
- [ ] Created CLAUDE.md for project standards
- [ ] Set up pre-edit hook script
- [ ] Configured .gitignore properly
- [ ] Tested configuration with /config
- [ ] Verified hooks run before file changes

### 💻 Verify Your Setup

**Check file structure:**
```bash
tree .claude
```

**Expected output:**
```
.claude/
├── CLAUDE.md
├── commands/
├── output-styles/
├── scripts/
│   └── pre-edit-check.sh
├── settings.json
└── settings.local.json
```

**Check what's in git:**
```bash
git status
```

**Should show:**
- ✅ `.claude/settings.json` (tracked)
- ✅ `.claude/CLAUDE.md` (tracked)
- ✅ `.claude/scripts/` (tracked)
- ❌ `.claude/settings.local.json` (ignored)

### 🎯 Challenge: Add Environment-Specific Configs

Create separate configs for dev/staging/prod:

<details>
<summary>💡 Solution</summary>

```bash
# Development (current settings.json is dev)
cp .claude/settings.json .claude/settings.dev.json

# Staging
cat > .claude/settings.staging.json << 'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash", "WebFetch"]
  },
  "env": {
    "ENVIRONMENT": "staging",
    "LOG_LEVEL": "info"
  }
}
EOF

# Production (read-only)
cat > .claude/settings.prod.json << 'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch"]
  },
  "env": {
    "ENVIRONMENT": "production",
    "LOG_LEVEL": "warning"
  }
}
EOF

# Use with:
# export CLAUDE_CONFIG=".claude/settings.prod.json"
# claude
```

</details>

**Key Insight:** Proper .claude configuration is like CI/CD for AI assistance - it ensures consistency, security, and compliance across your team.

---

### What to Commit vs Ignore

**Commit to Git (Team-Shared):**
- `.claude/settings.json` - Team configuration
- `.claude/CLAUDE.md` - Project memory and standards
- `.claude/commands/` - Custom slash commands
- `.claude/output-styles/` - Custom output styles

**Add to .gitignore (Personal):**
- `.claude/settings.local.json` - Personal preferences
- `.claude/.sessions/` - Session history (if exists)

**Example .gitignore:**
```gitignore
# Claude Code personal settings
.claude/settings.local.json
.claude/.sessions/
```

---

## Settings Hierarchy

Claude Code uses a **layered configuration system** with clear precedence rules.

### Configuration Precedence (Highest to Lowest)

```
1. Enterprise Managed Policies  (System-wide, IT managed)
   ↓
2. Command Line Arguments       (Session-specific)
   ↓
3. Local Project Settings       (.claude/settings.local.json)
   ↓
4. Shared Project Settings      (.claude/settings.json)
   ↓
5. User Settings                (~/.claude/settings.json)
```

**Higher priority settings override lower priority settings.**

### Configuration Locations

| Level | Path | Scope | Managed By | Committed |
|-------|------|-------|------------|-----------|
| **Enterprise** | `/Library/Application Support/ClaudeCode/` or system-wide | All users, all projects | IT/DevOps | N/A |
| **User** | `~/.claude/settings.json` | All projects for this user | Individual user | No |
| **Project (Shared)** | `./.claude/settings.json` | This project, all team members | Team | Yes |
| **Project (Local)** | `./.claude/settings.local.json` | This project, this user only | Individual user | No |
| **CLI** | Command flags | This session only | User | N/A |

### Example: Permission Override

```
Enterprise says: "No auto-approve mode"
User settings say: "permission-mode: auto-approve" → BLOCKED

Project settings say: "Allow Read, Grep only"
CLI flag says: "--permission-mode plan" → CLI WINS
```

---

## settings.json (Team-Shared)

The `.claude/settings.json` file contains team-wide configuration.

### Basic Structure

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit", "Write"],
    "deny": []
  },
  "defaultModel": "sonnet",
  "hooks": {},
  "env": {},
  "plugins": []
}
```

### Complete Example

```json
{
  "$schema": "https://claude.ai/schemas/settings.json",

  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Edit",
      "Write",
      "Bash"
    ],
    "deny": [
      "WebFetch",
      "WebSearch"
    ],
    "requireApproval": [
      "Write",
      "Edit",
      "Bash"
    ]
  },

  "defaultModel": "sonnet",

  "env": {
    "PYTHON_ENV": "development",
    "API_ENDPOINT": "https://api-dev.bank.internal",
    "LOG_LEVEL": "debug"
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/pre-edit-check.sh"
          }
        ]
      }
    ]
  },

  "outputStyle": "default",

  "context": {
    "maxTokens": 200000,
    "includePatterns": ["src/**", "tests/**"],
    "excludePatterns": ["__pycache__/**", "*.log", "*.pyc"]
  }
}
```

### Banking Example: Restrictive Settings

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob"
    ],
    "deny": [
      "Bash",
      "WebFetch",
      "WebSearch"
    ],
    "requireApproval": [
      "Edit",
      "Write"
    ]
  },

  "defaultModel": "sonnet",

  "env": {
    "ENVIRONMENT": "production-readonly",
    "COMPLIANCE_MODE": "strict"
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/audit-log.sh",
            "description": "Log all tool usage for compliance"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/security-scan.sh",
            "description": "Scan changes for security issues"
          }
        ]
      }
    ]
  }
}
```

---

## settings.local.json (Personal)

The `.claude/settings.local.json` file contains **personal preferences** that override project settings.

### Purpose

- Personal preferences (e.g., preferred model)
- Local development environment variables
- Personal shortcuts and commands
- Override team settings for your local workflow

### Example

```json
{
  "defaultModel": "sonnet",

  "env": {
    "MY_LOCAL_API_KEY": "dev-key-12345",
    "DEBUG": "true"
  },

  "permissions": {
    "allow": ["Bash"]
  },

  "outputStyle": "detailed"
}
```

### When to Use settings.local.json

**Good use cases:**
```json
{
  // Personal preference: Explicit model selection
  "defaultModel": "sonnet",

  // Local development secrets
  "env": {
    "LOCAL_DB_PASSWORD": "dev-password"
  },

  // Enable more tools for your development
  "permissions": {
    "allow": ["Bash", "WebFetch"]
  }
}
```

**Bad use cases (use settings.json instead):**
```json
{
  // DON'T: Team coding standards belong in settings.json
  "codeStyle": "pep8",

  // DON'T: Shared environment config belongs in settings.json
  "env": {
    "API_ENDPOINT": "https://api.company.com"
  }
}
```

---

## Permission Configuration

Fine-grained control over what Claude can do.

### Permission Model

```json
{
  "permissions": {
    "allow": ["tool1", "tool2"],
    "deny": ["tool3"],
    "requireApproval": ["tool4"]
  }
}
```

**Precedence:**
1. `deny` - Always blocked, highest priority
2. `allow` - Allowed without approval
3. `requireApproval` - Allowed but requires approval
4. Default - If not specified, requires approval

### Available Tools

| Tool | Purpose | Risk Level |
|------|---------|------------|
| `Read` | Read files | Low |
| `Grep` | Search files | Low |
| `Glob` | Find files | Low |
| `Write` | Create new files | Medium |
| `Edit` | Modify existing files | Medium |
| `Bash` | Execute shell commands | High |
| `WebFetch` | Fetch URLs | Medium |
| `WebSearch` | Search the web | Medium |
| `Task` | Launch sub-agents | Low |
| `TodoWrite` | Manage task list | Low |
| `NotebookEdit` | Edit Jupyter notebooks | Medium |

### Permission Profiles

#### Developer (Permissive)

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Edit",
      "Write",
      "Bash",
      "Task",
      "TodoWrite"
    ],
    "deny": [],
    "requireApproval": [
      "Bash",
      "Edit",
      "Write"
    ]
  }
}
```

#### Reviewer (Read-Only)

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Task"
    ],
    "deny": [
      "Edit",
      "Write",
      "Bash",
      "WebFetch",
      "WebSearch"
    ]
  }
}
```

#### Security Auditor (Restricted)

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob"
    ],
    "deny": [
      "Edit",
      "Write",
      "Bash",
      "WebFetch",
      "WebSearch",
      "NotebookEdit"
    ]
  }
}
```

### Banking Example: Role-Based Permissions

**Junior Developer:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash", "WebFetch"]
  }
}
```

**Senior Developer:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit", "Write", "Task"],
    "requireApproval": ["Bash"],
    "deny": ["WebFetch", "WebSearch"]
  }
}
```

**DevOps Engineer:**
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

---

## Environment Variables

Define environment variables for your project.

### Basic Configuration

```json
{
  "env": {
    "PYTHON_ENV": "development",
    "API_URL": "https://api-dev.company.com",
    "LOG_LEVEL": "debug",
    "DATABASE_URL": "postgresql://localhost:5432/devdb"
  }
}
```

### Variable Expansion

Claude Code supports variable expansion:

```json
{
  "env": {
    "HOME_DIR": "$HOME",
    "PROJECT_ROOT": "${PWD}",
    "CONFIG_PATH": "${HOME}/.config/app",
    "COMBINED": "${API_URL}/api/v1"
  }
}
```

### Banking Example: Environment-Specific Config

**Development (.claude/settings.local.json):**
```json
{
  "env": {
    "ENVIRONMENT": "development",
    "API_ENDPOINT": "https://api-dev.bank.internal",
    "DB_HOST": "localhost",
    "DB_PORT": "5432",
    "DB_NAME": "banking_dev",
    "LOG_LEVEL": "debug",
    "ENABLE_DEBUG_LOGGING": "true",
    "MOCK_EXTERNAL_APIS": "true",
    "SPARK_HOME": "/opt/spark",
    "PYSPARK_PYTHON": "python3",
    "PYSPARK_DRIVER_PYTHON": "python3"
  }
}
```

**Production (.claude/settings.json - for read-only access):**
```json
{
  "env": {
    "ENVIRONMENT": "production",
    "API_ENDPOINT": "https://api.bank.com",
    "LOG_LEVEL": "info",
    "ENABLE_DEBUG_LOGGING": "false",
    "READ_ONLY_MODE": "true",
    "SPARK_HOME": "/opt/spark",
    "PYSPARK_PYTHON": "python3"
  }
}
```

### Secrets Management

**⚠️ Never commit secrets to .claude/settings.json!**

**Good - Use environment variables:**
```json
{
  "env": {
    "API_KEY": "${BANKING_API_KEY}",
    "DB_PASSWORD": "${DB_PASSWORD}"
  }
}
```

Then set in your shell:
```bash
export BANKING_API_KEY="your-key-here"
export DB_PASSWORD="your-password-here"
```

**Bad - Hardcoded secrets:**
```json
{
  "env": {
    "API_KEY": "sk-1234567890",  // DON'T DO THIS!
    "DB_PASSWORD": "mypassword"  // NEVER COMMIT THIS!
  }
}
```

---

## Output Styles

Customize how Claude presents information and code.

### What are Output Styles?

Output styles control Claude's:
- Tone and communication style
- Level of detail in explanations
- How it approaches tasks
- Whether it provides educational context

### Built-in Styles

| Style | Description | Best For |
|-------|-------------|----------|
| `default` | Standard software engineering assistant | General development |
| `explanatory` | Provides "Insights" during tasks | Learning, onboarding |
| `learning` | Collaborative mode with TODO markers | Pair programming, teaching |

### Selecting Output Style

In settings.json:
```json
{
  "outputStyle": "explanatory"
}
```

Or change during session:
```bash
> /output-style explanatory
```

### Custom Output Styles

Create `.claude/output-styles/detailed.md`:

```markdown
---
name: detailed
description: Provides extensive documentation and explanations
---

# Detailed Output Style

You are an assistant that provides:
- Comprehensive explanations for every action
- Step-by-step reasoning
- Documentation for all code changes
- Links to relevant documentation
- Examples of usage

When writing code:
- Add extensive comments
- Include comprehensive docstrings (Google/NumPy style)
- Provide usage examples
- Explain trade-offs and design decisions
- Add type hints (PEP 484)

When reviewing code:
- Provide detailed feedback
- Suggest improvements with reasoning
- Reference best practices (PEP 8, PEP 20)
- Include code examples
```

Use it:
```json
{
  "outputStyle": "detailed"
}
```

### Banking Example: Compliance-Focused Style

`.claude/output-styles/compliance.md`:
```markdown
---
name: compliance
description: Focuses on regulatory compliance and security
---

# Compliance-Focused Assistant

You are a banking compliance-focused assistant for Python/PySpark data engineering.

## Code Generation
- Always consider PCI-DSS, SOX, and GDPR requirements
- Include audit logging by default
- Add security checks and validation
- Document compliance requirements in docstrings
- Use type hints for data validation
- Implement proper error handling with logging

## Code Review
- Flag potential compliance issues
- Check for sensitive data exposure
- Verify audit logging
- Ensure proper error handling
- Check authentication/authorization
- Verify PII data masking in logs
- Check for SQL injection vulnerabilities in dynamic queries

## Documentation
- Include compliance notes in docstrings
- Document data handling procedures
- Note security considerations
- Reference relevant regulations
- Document data lineage and transformations
```

---

## Banking IT Templates

Ready-to-use templates for banking projects.

### Template 1: Standard Development Project

`.claude/settings.json`:
```json
{
  "$schema": "https://claude.ai/schemas/settings.json",

  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit", "Write", "Task", "TodoWrite"],
    "deny": ["WebSearch"],
    "requireApproval": ["Bash", "Edit", "Write", "WebFetch"]
  },

  "defaultModel": "sonnet",

  "env": {
    "ENVIRONMENT": "development",
    "LOG_LEVEL": "debug",
    "API_URL": "https://api-dev.bank.internal",
    "SPARK_HOME": "/opt/spark",
    "PYSPARK_PYTHON": "python3"
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix",
            "description": "Run linter before changes"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/audit-log.sh \"${TOOL_NAME}\" \"${FILE_PATH}\"",
            "description": "Log file changes for audit"
          }
        ]
      }
    ]
  },

  "outputStyle": "default"
}
```

### Template 2: Production Read-Only Access

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch"]
  },

  "defaultModel": "sonnet",

  "env": {
    "ENVIRONMENT": "production",
    "READ_ONLY": "true"
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/audit-log.sh",
            "description": "Log all access for compliance"
          }
        ]
      }
    ]
  }
}
```

### Template 3: Security-Focused Review

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch"]
  },

  "defaultModel": "sonnet",

  "env": {
    "REVIEW_MODE": "security",
    "COMPLIANCE_STANDARDS": "PCI-DSS,SOX,GDPR"
  },

  "outputStyle": "compliance"
}
```

---

## Best Practices

### 1. Version Control Settings

**Commit to Git (Execute Manually):**
```bash
# Execute these commands manually (banking IT policy)
git add .claude/settings.json
git add .claude/CLAUDE.md
git add .claude/commands/
git commit -m "Add Claude Code configuration"
```

**Note**: Per banking IT policy, all git operations must be executed manually by developers. Claude Code does NOT execute git commands.

**Add to .gitignore:**
```gitignore
.claude/settings.local.json
.claude/.sessions/
```

### 2. Document Your Configuration

Add a comment header to settings.json:
```json
{
  "_comment": "Banking ETL Project - Claude Code Configuration",
  "_description": "Team-wide settings for Claude Code. See README.md for details.",
  "_lastUpdated": "2025-10-23",
  "_owner": "DevOps Team",

  "permissions": {
    // ... configuration
  }
}
```

### 3. Start Restrictive, Then Loosen

```json
// Start with minimal permissions
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  }
}

// Add permissions as needed by team
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit"],
    "requireApproval": ["Write", "Bash"],
    "deny": []
  }
}
```

### 4. Use Environment-Specific Config Files

```bash
.claude/
├── settings.json                    # Base settings
├── settings.development.json        # Dev overrides
├── settings.staging.json            # Staging overrides
└── settings.production.json         # Prod overrides
```

Switch via environment:
```bash
# Development
export CLAUDE_CONFIG=".claude/settings.development.json"
claude

# Production
export CLAUDE_CONFIG=".claude/settings.production.json"
claude --permission-mode plan
```

### 5. Regular Configuration Audits

```bash
# Review settings quarterly
> /config

# Check for:
# - Overly permissive settings
# - Outdated environment variables
# - Unused hooks
# - Security issues
```

### 6. Team Communication

Create `.claude/README.md`:
```markdown
# Claude Code Configuration

## Overview
This project uses Claude Code for development assistance.

## Settings
- `settings.json`: Team-wide configuration (committed)
- `settings.local.json`: Personal overrides (not committed)

## Permissions
- Read/Grep/Glob: Allowed
- Edit/Write: Requires approval
- Bash: Restricted to DevOps team

## Custom Commands
- `/review`: Code review with security focus
- `/test`: Generate tests
- `/doc`: Generate documentation

## Contact
Questions? Ask #devops-team on Slack
```

---

## Summary

In this subsection, you learned:

### Core Concepts
- ✅ `.claude/` directory structure
- ✅ Settings hierarchy and precedence
- ✅ Team-shared vs personal configuration

### Configuration Files
- ✅ `settings.json` for team configuration
- ✅ `settings.local.json` for personal preferences
- ✅ Permission models and tool restrictions

### Advanced Features
- ✅ Environment variable configuration
- ✅ Output styles customization
- ✅ Role-based permission templates

### Banking IT Applications
- ✅ Security-focused configurations
- ✅ Compliance-aware settings
- ✅ Role-based access control
- ✅ Audit logging integration

---

## Next Steps

👉 **[Continue to 1.3.6: Slash Commands](./03-06-slash-commands.md)**

**Quick Practice:**
1. Create `.claude/settings.json` with dev permissions
2. Set up environment variables
3. Create a custom output style

---

**Related Sections:**
- [Phase 1.3.2: Security & Permissions](./03-02-security-permissions.md) - Permission details
- [Phase 1.3.3: Memory & Context](./03-03-memory-context.md) - CLAUDE.md
- [Phase 1.3.8: Hooks & Automation](./03-08-hooks-automation.md) - Hook configuration

---

**Last Updated:** 2025-10-24
**Version:** 1.0
