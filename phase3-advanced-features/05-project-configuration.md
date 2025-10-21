# Section 5: Project Configuration (.claude Directory)

## Table of Contents
1. [The .claude Directory](#the-claude-directory)
2. [Settings Hierarchy](#settings-hierarchy)
3. [settings.json (Team-Shared)](#settingsjson-team-shared)
4. [settings.local.json (Personal)](#settingslocaljson-personal)
5. [Permission Configuration](#permission-configuration)
6. [Environment Variables](#environment-variables)
7. [Output Styles](#output-styles)
8. [Plugin Configuration](#plugin-configuration)
9. [Banking IT Templates](#banking-it-templates)
10. [Best Practices](#best-practices)

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
cd ~/projects/banking-api

# Create .claude directory structure
mkdir -p .claude/commands .claude/output-styles

# Create settings files
touch .claude/settings.json
touch .claude/settings.local.json
touch .claude/CLAUDE.md

# Add to .gitignore
echo ".claude/settings.local.json" >> .gitignore
```

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
    "NODE_ENV": "development",
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
    "excludePatterns": ["node_modules/**", "*.log"]
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
  "defaultModel": "opus",

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
  // Personal preference: Use Opus locally
  "defaultModel": "opus",

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
  "codeStyle": "airbnb",

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
    "NODE_ENV": "development",
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
    "MOCK_EXTERNAL_APIS": "true"
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
    "READ_ONLY_MODE": "true"
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
- Include JSDoc/docstrings
- Provide usage examples
- Explain trade-offs and design decisions

When reviewing code:
- Provide detailed feedback
- Suggest improvements with reasoning
- Reference best practices
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

You are a banking compliance-focused assistant.

## Code Generation
- Always consider PCI-DSS, SOX, and GDPR requirements
- Include audit logging by default
- Add security checks and validation
- Document compliance requirements in comments

## Code Review
- Flag potential compliance issues
- Check for sensitive data exposure
- Verify audit logging
- Ensure proper error handling
- Check authentication/authorization

## Documentation
- Include compliance notes
- Document data handling procedures
- Note security considerations
- Reference relevant regulations
```

---

## Plugin Configuration

Extend Claude Code with plugins (if available in your version).

### Plugin Structure

```json
{
  "plugins": [
    {
      "name": "eslint-checker",
      "enabled": true,
      "config": {
        "autoFix": false,
        "rules": "recommended"
      }
    },
    {
      "name": "prettier-formatter",
      "enabled": true,
      "config": {
        "printWidth": 100,
        "singleQuote": true
      }
    }
  ]
}
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
    "API_URL": "https://api-dev.bank.internal"
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint-staged",
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

  "defaultModel": "opus",

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

  "defaultModel": "opus",

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

**Commit to Git:**
```bash
git add .claude/settings.json
git add .claude/CLAUDE.md
git add .claude/commands/
git commit -m "Add Claude Code configuration"
```

**Add to .gitignore:**
```gitignore
.claude/settings.local.json
.claude/.sessions/
```

### 2. Document Your Configuration

Add a comment header to settings.json:
```json
{
  "_comment": "Banking API Project - Claude Code Configuration",
  "_description": "Team-wide settings for Claude Code. See README.md for details.",
  "_lastUpdated": "2025-10-19",
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

In this section, you learned:

### Core Concepts
- `.claude/` directory structure
- Settings hierarchy and precedence
- Team-shared vs personal configuration

### Configuration Files
- `settings.json` for team configuration
- `settings.local.json` for personal preferences
- Permission models and tool restrictions

### Advanced Features
- Environment variable configuration
- Output styles customization
- Plugin integration
- Role-based permission templates

### Banking IT Applications
- Security-focused configurations
- Compliance-aware settings
- Role-based access control
- Audit logging integration

---

## Next Steps

1. **[Continue to Section 6: Memory Management](./06-memory-management.md)** - Configure CLAUDE.md files
2. **[Jump to Section 9: Security & Compliance](../phase4-enterprise-security/09-security-compliance.md)** - Security deep dive
3. **[Review Settings Documentation](https://docs.claude.com/en/docs/claude-code/settings)** - Official settings reference

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-4

**Official References:**
- **Settings Documentation**: https://docs.claude.com/en/docs/claude-code/settings
- **Configuration Guide**: https://docs.claude.com/en/docs/claude-code/configuration
- **Output Styles**: https://docs.claude.com/en/docs/claude-code/output-styles
