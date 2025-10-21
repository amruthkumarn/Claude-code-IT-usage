# Section 13: Standards & Best Practices

## Table of Contents
1. [Organizational Standards](#organizational-standards)
2. [Project Setup](#project-setup)
3. [Team Collaboration](#team-collaboration)
4. [Security Best Practices](#security-best-practices)
5. [Performance Optimization](#performance-optimization)
6. [Banking-Specific Standards](#banking-specific-standards)

---

## Organizational Standards

### Naming Conventions

**Custom Slash Commands:**
```
.claude/commands/
├── compliance-check.md      # Clear, descriptive
├── security-review.md        # Hyphenated
├── api-docs.md              # Abbreviations OK if common
└── generate-tests.md        # Verb-noun format
```

**Memory Files:**
```
CLAUDE.md                    # Main project memory
.claude/CLAUDE.md           # Alternative location
~/.claude/CLAUDE.md         # User memory
```

**Configuration Files:**
```
.claude/settings.json        # Team-shared
.claude/settings.local.json  # Personal (gitignored)
```

### File Organization

```
your-project/
├── .claude/
│   ├── settings.json              # Team configuration
│   ├── settings.local.json        # Personal (in .gitignore)
│   ├── CLAUDE.md                  # Project memory
│   ├── commands/                  # Custom slash commands
│   │   ├── README.md              # Command documentation
│   │   ├── review/                # Organized by category
│   │   │   ├── security.md
│   │   │   └── compliance.md
│   │   └── generate/
│   │       ├── tests.md
│   │       └── docs.md
│   ├── hooks/                     # Hook scripts
│   │   ├── README.md
│   │   ├── audit-log.sh
│   │   └── security-check.sh
│   └── output-styles/             # Custom output styles
│       └── detailed.md
├── .gitignore                     # Include .claude/settings.local.json
└── README.md                      # Document Claude Code usage
```

---

## Project Setup

### Initial Setup Checklist

**1. Create .claude directory:**
```bash
mkdir -p .claude/{commands,hooks,output-styles}
```

**2. Create settings.json:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": []
  },
  "defaultModel": "sonnet",
  "hooks": {}
}
```

**3. Create CLAUDE.md:**
```markdown
# Project Name

## Coding Standards
[Your standards]

## Security Requirements
[Your requirements]

## Testing Standards
[Your testing approach]
```

**4. Update .gitignore:**
```gitignore
# Claude Code
.claude/settings.local.json
.claude/.sessions/
```

**5. Create README section:**
```markdown
## Claude Code Setup

This project uses Claude Code for AI-assisted development.

### Getting Started
1. Install: `npm install -g @anthropic-ai/claude-code`
2. Login: `claude /login`
3. Start: `claude`

### Custom Commands
- `/compliance-check` - Check PCI-DSS/SOX compliance
- `/security-review` - Security audit
- `/generate-tests` - Generate test cases

See `.claude/commands/README.md` for full list.
```

---

## Team Collaboration

### Shared Configuration

**What to commit:**
- `.claude/settings.json` - Team settings
- `.claude/CLAUDE.md` - Project standards
- `.claude/commands/` - Custom commands
- `.claude/hooks/` - Hook scripts
- `.claude/output-styles/` - Custom styles

**What NOT to commit:**
- `.claude/settings.local.json` - Personal settings
- `.claude/.sessions/` - Session history

### Code Review with Claude

**Reviewer checklist:**
```markdown
# Code Review Checklist

Before approving PR:
- [ ] Run `/security-review` on changed files
- [ ] Run `/compliance-check` for regulatory code
- [ ] Verify test coverage with `/test-coverage`
- [ ] Check for secrets with `/detect-secrets`
- [ ] Review Claude Code audit log for this session
```

### Onboarding New Team Members

**Day 1 - Setup:**
```markdown
1. Install Claude Code
2. Clone project
3. Run `claude` in project directory
4. Try `/help` to see custom commands
5. Read `.claude/CLAUDE.md` for standards
```

**Day 2 - Practice:**
```markdown
1. Use Plan Mode to explore: `claude --permission-mode plan`
2. Ask questions about the codebase
3. Try fixing a small bug with Claude's help
4. Review changes carefully before approving
```

---

## Security Best Practices

### 1. Principle of Least Privilege

**Development:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  }
}
```

**Production (Read-Only):**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch"]
  }
}
```

### 2. Secrets Management

**Never:**
```json
{
  "env": {
    "API_KEY": "sk-1234567890"  // NEVER hardcode secrets!
  }
}
```

**Always:**
```json
{
  "env": {
    "API_KEY": "${API_KEY}"  // Reference environment variable
  }
}
```

### 3. Audit Logging

Enable for all environments:
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

### 4. Regular Security Reviews

```bash
# Monthly security audit
claude --permission-mode plan

> @security-auditor perform comprehensive security review

> Check for:
> - SQL injection vulnerabilities
> - Hardcoded secrets
> - Authentication bypasses
> - Sensitive data exposure
```

---

## Performance Optimization

### 1. Use Available Model

```bash
# Sonnet is currently the only available model in AWS Bedrock
# Suitable for all development tasks: quick queries, development, complex analysis
claude

# Or explicitly specify
claude --model sonnet

# For read-only analysis
claude --permission-mode plan
> Analyze the architecture and suggest improvements
```

**Note:** Only Sonnet is currently available in AWS Bedrock. It provides excellent performance for all development tasks.

### 2. Manage Context

**Keep sessions focused:**
```bash
# Don't:
> Tell me about file1.ts
> Now file2.ts
> Now file3.ts
# ... (context bloat)

# Do:
> Explain the authentication system (files: auth/*.ts)
```

**Start fresh when needed:**
```bash
# If Claude "forgets" things
Ctrl+D  # Exit
claude  # New session, clean context
```

### 3. Use Agents for Subtasks

```bash
# Instead of one long session:
> @security-auditor review security
> @test-generator create tests
> @doc-writer generate docs

# Each agent has clean context
```

---

## Banking-Specific Standards

### Code Review Standards

```markdown
# Banking Code Review Checklist

## Security
- [ ] No hardcoded credentials
- [ ] SQL queries use parameterized statements
- [ ] Authentication required on endpoints
- [ ] Input validation implemented
- [ ] Error messages don't leak info

## Compliance
- [ ] PCI-DSS: No card data in logs
- [ ] SOX: Audit trail for financial transactions
- [ ] GDPR: Personal data properly handled
- [ ] No CVV storage anywhere

## Quality
- [ ] Test coverage ≥ 80%
- [ ] Code follows style guide
- [ ] Documentation updated
- [ ] No console.log in production code
```

### Deployment Standards

```markdown
## Pre-Deployment Checklist

- [ ] All tests passing
- [ ] Security scan completed (no high/critical issues)
- [ ] Compliance check passed
- [ ] Code review approved (2+ reviewers)
- [ ] Database migrations tested
- [ ] Rollback plan documented
- [ ] Monitoring alerts configured
- [ ] Runbook updated
```

### Incident Response

```markdown
## Using Claude Code in Incidents

1. **Investigation (Read-Only)**
   ```bash
   claude --permission-mode plan
   > Analyze error logs in /var/log/app/
   > Search for similar issues in codebase
   ```

2. **Fix Development (Dev Environment)**
   ```bash
   git checkout -b hotfix/issue-123
   claude
   > Fix the identified issue
   > Add tests to prevent regression
   ```

3. **Documentation**
   ```bash
   > Generate incident report with timeline and root cause analysis
   ```

**Never:** Use Claude Code directly in production during incidents!
```

---

## Summary

In this section, you learned:

### Organization
- Naming conventions for commands and files
- Project structure best practices
- What to commit vs. gitignore

### Team Collaboration
- Onboarding process
- Shared configuration
- Code review workflows

### Security
- Least privilege principle
- Secrets management
- Audit logging
- Regular security reviews

### Performance
- Model selection strategy
- Context management
- Using agents effectively

### Banking Standards
- Code review checklists
- Deployment standards
- Incident response guidelines

---

## Next Steps

1. **[Continue to Section 14: Templates Library](../06-reference/14-templates-library.md)** - Ready-to-use templates
2. **Implement these standards** - Adapt for your team
3. **Create your checklist** - Customize for your needs

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-12
