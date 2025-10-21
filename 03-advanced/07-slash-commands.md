# Section 7: Slash Commands

## Table of Contents
1. [What are Slash Commands?](#what-are-slash-commands)
2. [Built-in Slash Commands](#built-in-slash-commands)
3. [Creating Custom Commands](#creating-custom-commands)
4. [Command Arguments](#command-arguments)
5. [Banking IT Command Examples](#banking-it-command-examples)
6. [Best Practices](#best-practices)

---

## What are Slash Commands?

**Slash commands** are special commands starting with `/` that trigger predefined actions or prompts in Claude Code.

### Types of Slash Commands

1. **Built-in Commands** - Provided by Claude Code (`/help`, `/clear`, etc.)
2. **Custom Project Commands** - Team-shared (`.claude/commands/`)
3. **Custom User Commands** - Personal (`~/.claude/commands/`)
4. **Plugin Commands** - From installed plugins
5. **MCP Commands** - From MCP servers

### Commands vs Regular Prompts

**Regular prompt:**
```
> Review this code for security issues and provide a detailed report
```

**Slash command:**
```
> /security-review
```

Both can achieve the same result, but commands are:
- Faster to type
- Consistent across sessions
- Shareable with team
- Can include complex logic

---

## Built-in Slash Commands

### Essential Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/clear` | Clear conversation history |
| `/exit` | Exit Claude Code |
| `/model [model]` | Change AI model (sonnet/opus/haiku) |
| `/config` | Open settings |
| `/login` | Login or check auth status |
| `/memory` | Edit memory files |
| `/review` | Request code review |
| `/output-style [style]` | Change output style |

### Usage Examples

```bash
# Get help
> /help

# Change model
> /model opus

# Clear history
> /clear

# Code review
> /review
```

---

## Creating Custom Commands

Custom commands are markdown files in `.claude/commands/` or `~/.claude/commands/`.

### Basic Command Structure

Create `.claude/commands/review.md`:

```markdown
---
description: Perform comprehensive code review
---

Review the code in this project for:
- Security vulnerabilities
- Performance issues
- Code quality problems
- Best practice violations
- Potential bugs

Provide a detailed report with specific file names and line numbers.
```

**Usage:**
```bash
> /review
```

### Command with Arguments

Create `.claude/commands/test.md`:

```markdown
---
description: Generate tests for a file
---

Generate comprehensive unit tests for: $1

Include:
- Happy path tests
- Edge cases
- Error conditions
- Mock external dependencies

Use Jest testing framework.
```

**Usage:**
```bash
> /test src/auth/login.ts
# $1 = "src/auth/login.ts"
```

### Command with Multiple Arguments

Create `.claude/commands/refactor.md`:

```markdown
---
description: Refactor code with specific pattern
---

Refactor the file $1 to use $2 pattern.

Target file: $1
Pattern: $2
Additional notes: $3

Ensure:
- Behavior remains unchanged
- Tests still pass
- Code is more maintainable
```

**Usage:**
```bash
> /refactor src/payment.js "async/await" "remove callbacks"
# $1 = src/payment.js
# $2 = async/await
# $3 = remove callbacks
```

### Command with All Arguments

Use `$ARGUMENTS` to get all arguments as a single string:

Create `.claude/commands/explain.md`:

```markdown
---
description: Explain a concept
---

Explain the following concept in the context of this codebase:

$ARGUMENTS

Provide:
1. Clear definition
2. How it's used in this project
3. Code examples from the codebase
4. Best practices
```

**Usage:**
```bash
> /explain dependency injection and why we use it
# $ARGUMENTS = "dependency injection and why we use it"
```

---

## Command Arguments

### Argument Syntax

| Syntax | Description | Example |
|--------|-------------|---------|
| `$1` | First argument | `/cmd arg1` → `$1 = "arg1"` |
| `$2` | Second argument | `/cmd arg1 arg2` → `$2 = "arg2"` |
| `$3` | Third argument | `/cmd a b c` → `$3 = "c"` |
| `$ARGUMENTS` | All arguments as string | `/cmd hello world` → `$ARGUMENTS = "hello world"` |

### Quoted Arguments

```bash
# Without quotes (space separates arguments)
> /refactor src/file.js async/await
# $1 = "src/file.js"
# $2 = "async/await"

# With quotes (treat as single argument)
> /refactor "src/file with spaces.js" "async/await pattern"
# $1 = "src/file with spaces.js"
# $2 = "async/await pattern"
```

---

## Banking IT Command Examples

### Example 1: Compliance Check

`.claude/commands/compliance-check.md`:

```markdown
---
description: Check code for banking compliance (PCI-DSS, SOX, GDPR)
---

Perform a comprehensive compliance check on: ${1:-.}

Check for:

## PCI-DSS Compliance
- Are credit card numbers being logged?
- Is sensitive data encrypted at rest and in transit?
- Are CVV codes being stored (must NOT be stored)?
- Is tokenization used for card data?

## SOX Compliance
- Are all financial transactions auditable?
- Is there proper separation of duties in code?
- Are database changes logged?

## GDPR Compliance
- Is personal data properly classified?
- Is there a data retention policy?
- Can users request data deletion?
- Is consent properly tracked?

## General Security
- SQL injection vulnerabilities?
- Hardcoded credentials?
- Proper authentication/authorization?

Provide a detailed report with:
- File paths and line numbers
- Severity (Critical/High/Medium/Low)
- Remediation steps
```

**Usage:**
```bash
> /compliance-check
# Checks entire project

> /compliance-check src/payments/
# Checks specific directory
```

### Example 2: Security Review

`.claude/commands/security-review.md`:

```markdown
---
description: Comprehensive security review
---

Perform security analysis on: ${1:-.}

## Authentication & Authorization
- Are endpoints properly protected?
- Is JWT validation correct?
- Are permissions checked?
- Session management secure?

## Input Validation
- All user inputs validated?
- SQL injection prevention?
- XSS prevention?
- CSRF protection?

## Data Protection
- Sensitive data encrypted?
- Secrets management proper?
- Password hashing secure (bcrypt/argon2)?
- API keys protected?

## Logging & Monitoring
- Security events logged?
- PII excluded from logs?
- Error messages don't leak info?

## Dependencies
- Known vulnerabilities?
- Outdated packages?
- Suspicious dependencies?

Provide actionable recommendations with code examples.
```

### Example 3: Generate API Documentation

`.claude/commands/api-docs.md`:

```markdown
---
description: Generate OpenAPI/Swagger documentation
---

Generate OpenAPI 3.0 documentation for: $1

Include:
- All endpoints (GET, POST, PUT, DELETE, PATCH)
- Request/response schemas
- Authentication requirements
- Error responses
- Example requests/responses
- Rate limiting information

Format as valid OpenAPI YAML.

Banking-specific requirements:
- Document all validation rules
- Include audit logging notes
- Specify compliance requirements
- Note transaction boundaries
```

**Usage:**
```bash
> /api-docs src/api/payments.ts
```

### Example 4: Database Migration Generator

`.claude/commands/migration.md`:

```markdown
---
description: Generate database migration
---

Generate a PostgreSQL migration for: $ARGUMENTS

Requirements:
- Use transaction for safety
- Include rollback (DOWN migration)
- Add indexes for foreign keys
- Include comments for clarity
- Follow bank naming conventions (snake_case)

Template format:
```sql
-- UP Migration
BEGIN;

-- Your changes here

COMMIT;

-- DOWN Migration (Rollback)
BEGIN;

-- Undo changes

COMMIT;
```

Include:
- Data type considerations
- Index creation
- Foreign key constraints
- NOT NULL constraints where appropriate
```

**Usage:**
```bash
> /migration add user_preferences table with user_id, theme, language
```

### Example 5: Test Coverage Report

`.claude/commands/coverage.md`:

```markdown
---
description: Analyze test coverage and generate missing tests
---

Analyze test coverage for: $1

1. Identify all functions/methods
2. Check which have tests
3. Calculate coverage percentage
4. Generate tests for untested code

Focus on:
- Edge cases
- Error conditions
- Banking-specific scenarios (overdraft, insufficient funds, etc.)
- Integration points
- Security-critical paths

Generate Jest tests following our standards:
- Descriptive test names
- Arrange-Act-Assert pattern
- Mock external dependencies
- Test isolation
```

### Example 6: Release Notes Generator

`.claude/commands/release-notes.md`:

```markdown
---
description: Generate release notes from git commits
---

Generate release notes for version: $1

Review git commits since last release and create:

## Features
- New functionality added

## Improvements
- Enhancements to existing features

## Bug Fixes
- Issues resolved

## Security
- Security patches and updates

## Breaking Changes
- Changes requiring migration

## Database Changes
- Schema migrations required

Format for:
- Internal team (technical details)
- Stakeholders (business impact)
- Compliance team (audit information)
```

**Usage:**
```bash
> /release-notes v2.5.0
```

---

## Best Practices

### 1. Descriptive Names

**Good:**
```
/security-review
/generate-tests
/compliance-check
```

**Bad:**
```
/sr
/gt
/check
```

### 2. Clear Descriptions

```markdown
---
description: Generate unit tests for a specific file
---
```

Not:
```markdown
---
description: Tests
---
```

### 3. Document Arguments

```markdown
---
description: Refactor code - Usage: /refactor <file> <pattern>
---

Refactor $1 to use $2 pattern.

Arguments:
- $1: File path to refactor
- $2: Pattern to apply (e.g., "async/await", "dependency-injection")
```

### 4. Provide Context

```markdown
---
description: Generate API endpoint
---

Generate a new REST API endpoint for: $ARGUMENTS

Use our project standards:
- Express.js framework
- JWT authentication
- Joi validation
- Async/await error handling
- OpenAPI documentation

Follow the pattern in src/api/example.ts
```

### 5. Include Examples

```markdown
---
description: Database query optimization
---

Optimize the database queries in: $1

Examples of optimizations:
1. Add indexes for frequently queried columns
2. Use EXPLAIN ANALYZE to identify slow queries
3. Implement query result caching
4. Batch multiple queries
5. Use connection pooling

Provide before/after performance estimates.
```

### 6. Team Organization

```
.claude/commands/
├── README.md                  # Command documentation
├── dev/                       # Development commands
│   ├── test.md
│   ├── lint.md
│   └── format.md
├── review/                    # Code review commands
│   ├── security-review.md
│   ├── compliance-check.md
│   └── performance-review.md
├── generate/                  # Code generation
│   ├── api-endpoint.md
│   ├── test-suite.md
│   └── migration.md
└── docs/                      # Documentation
    ├── api-docs.md
    └── release-notes.md
```

### 7. Version Control

```bash
# Execute these commands manually (banking IT policy)
# Commit custom commands
git add .claude/commands/
git commit -m "docs: add custom Claude Code commands for team"

# Document in README
echo "## Claude Code Commands" >> README.md
echo "Run \`/help\` to see available custom commands" >> README.md
```

**Note**: Per banking IT policy, execute all git commands manually. Claude Code does NOT run git operations.

---

## Summary

In this section, you learned:

### Core Concepts
- Slash commands for quick, repeatable tasks
- Built-in vs custom commands
- Project vs user commands

### Implementation
- Creating custom commands with markdown
- Using arguments ($1, $2, $ARGUMENTS)
- Frontmatter for metadata

### Banking Applications
- Compliance checking commands
- Security review automation
- API documentation generation
- Database migration helpers
- Release notes automation

---

## Next Steps

1. **[Continue to Section 8: Agents & Sub-agents](./08-agents-subagents.md)** - Specialized AI agents
2. **[Review Slash Commands Docs](https://docs.claude.com/en/docs/claude-code/slash-commands)** - Official reference
3. **Create your first custom command** - Start with a simple `/review` command

---


**Official References:**
- **Slash Commands**: https://docs.claude.com/en/docs/claude-code/slash-commands
- **Custom Commands Guide**: https://docs.claude.com/en/docs/claude-code/custom-commands
