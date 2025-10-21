# Section 6: Memory Management

## Table of Contents
1. [Understanding Memory in Claude Code](#understanding-memory-in-claude-code)
2. [The CLAUDE.md File](#the-claudemd-file)
3. [Memory Hierarchy](#memory-hierarchy)
4. [Writing Effective Memory](#writing-effective-memory)
5. [Memory Management Commands](#memory-management-commands)
6. [Banking IT Memory Examples](#banking-it-memory-examples)
7. [Best Practices](#best-practices)

---

## Understanding Memory in Claude Code

**Memory** in Claude Code refers to persistent instructions that Claude reads at the start of every session. Unlike conversation context (which is temporary), memory persists across all sessions.

### What is Memory?

Memory is stored in **CLAUDE.md** files (markdown format) that contain:
- Coding standards and style guides
- Project-specific conventions
- Security requirements
- Common patterns and best practices
- Things Claude should always remember

### Why Use Memory?

Without memory:
```
Session 1:
> Use TypeScript for all new code
Claude: OK, I'll use TypeScript

Session 2 (new day):
> Add a new feature
Claude: [generates JavaScript code]
```

With memory (in CLAUDE.md):
```markdown
# Always use TypeScript for new code
```

Now Claude remembers this across ALL sessions automatically!

---

## The CLAUDE.md File

### File Format

CLAUDE.md uses standard Markdown syntax:

```markdown
# Project Name

## Coding Standards
- Use TypeScript
- Follow Airbnb style guide
- Maximum line length: 100 characters

## Security Requirements
- All API endpoints must require authentication
- Never log sensitive data (passwords, tokens, SSNs)
- Use parameterized queries for SQL

## Testing Standards
- Minimum 80% code coverage
- Write tests before implementation (TDD)
- Mock external dependencies
```

### Location Options

| Location | Scope | Committed | Purpose |
|----------|-------|-----------|---------|
| `/Library/Application Support/ClaudeCode/CLAUDE.md` | Enterprise-wide | System | IT-mandated policies |
| `~/.claude/CLAUDE.md` | User (all projects) | No | Personal preferences |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project (team) | Yes | Project standards |

---

## Memory Hierarchy

Memory files are loaded in a specific order, with **higher levels taking precedence**:

```
┌────────────────────────────────────────┐
│  1. Enterprise Policy Memory           │  ← Highest Priority
│     (System-wide, IT managed)          │    Cannot be overridden
│  /Library/.../ClaudeCode/CLAUDE.md    │
└────────────────┬───────────────────────┘
                 │
┌────────────────▼───────────────────────┐
│  2. Project Memory                     │
│     (Team-shared, in git)              │
│  ./CLAUDE.md or ./.claude/CLAUDE.md   │
└────────────────┬───────────────────────┘
                 │
┌────────────────▼───────────────────────┐
│  3. User Memory                        │  ← Lowest Priority
│     (Personal preferences)             │
│  ~/.claude/CLAUDE.md                  │
└────────────────────────────────────────┘
```

### Precedence Rules

**Enterprise memory overrides everything:**
```markdown
# /Library/.../ClaudeCode/CLAUDE.md
NEVER use auto-approve mode
```

Even if user/project memory says otherwise, this rule applies.

**Project memory overrides user memory:**
```markdown
# Project: ./CLAUDE.md
Use 4 spaces for indentation

# User: ~/.claude/CLAUDE.md
Use 2 spaces for indentation
```

Claude will use 4 spaces (project wins).

---

## Writing Effective Memory

### Structure Your Memory

Use clear markdown structure:

```markdown
# Project Name / Purpose

Brief description of the project.

## Coding Standards

### TypeScript
- Use strict mode
- Define interfaces for all public APIs
- Prefer interfaces over types

### Code Style
- Use ESLint with Airbnb config
- Prettier for formatting
- Max line length: 100

## Architecture

### Directory Structure
```
src/
  api/       - API endpoints
  services/  - Business logic
  models/    - Data models
  utils/     - Utility functions
```

### Design Patterns
- Use dependency injection
- Follow SOLID principles
- Repository pattern for data access

## Security

### Authentication
- All endpoints require JWT token
- Token expiry: 1 hour
- Refresh token: 7 days

### Data Handling
- Never log: passwords, tokens, SSNs, credit cards
- Encrypt PII at rest
- Use HTTPS only

## Testing
- Unit tests: Jest
- Integration tests: Supertest
- E2E tests: Cypress
- Minimum coverage: 80%

## Git Workflow
- Feature branches: `feature/description`
- Commit messages: Conventional Commits
- Require PR reviews: 2 approvals
```

### Be Specific, Not Vague

**Vague:**
```markdown
- Write good code
- Follow best practices
- Make it secure
```

**Specific:**
```markdown
- Use async/await, never callbacks
- All database queries must use parameterized statements
- Validate all user input with Joi schemas
- Return 401 for authentication errors, 403 for authorization errors
```

### Use Examples

```markdown
## Error Handling

All API endpoints should use this error response format:

\`\`\`json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "User-friendly message",
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    }
  }
}
\`\`\`

Example implementation:
\`\`\`typescript
throw new AppError({
  code: 'INVALID_INPUT',
  message: 'Invalid email format',
  statusCode: 400,
  details: { field: 'email' }
});
\`\`\`
```

### Use Checklists

```markdown
## Pre-Deployment Checklist

Before deploying to production, verify:
- [ ] All tests passing
- [ ] Code coverage ≥ 80%
- [ ] No console.log statements
- [ ] Environment variables documented
- [ ] API documentation updated
- [ ] Database migrations tested
- [ ] Security scan completed
- [ ] Performance tested
```

---

## Memory Management Commands

### Viewing Memory

```bash
# Open memory file in editor
> /memory

# Shows all memory locations and allows editing
```

### Quick Memory Addition

```bash
# In Claude Code session, use # shortcut
> # Always use async/await for async operations

# This adds to current project's CLAUDE.md
```

### Importing External Memory

```markdown
# In CLAUDE.md, import other files:
@/path/to/shared-standards.md
@../team-conventions.md
```

Example:
```markdown
# Banking API Project

@~/.config/bank-coding-standards.md

## Project-Specific Rules
- Use Express.js for API
- PostgreSQL for database
```

---

## Banking IT Memory Examples

### Example 1: Payment Processing Service

`.claude/CLAUDE.md`:
```markdown
# Payment Processing Service

## Overview
Handles payment transactions for retail banking customers.

## Security Requirements (PCI-DSS Compliance)

### Critical Rules
- NEVER log credit card numbers (full or partial)
- NEVER store CVV codes
- All payment data must be encrypted in transit and at rest
- Use tokenization for card storage
- Log all payment attempts for audit

### Authentication
- All endpoints require OAuth2 bearer token
- Rate limiting: 100 requests per minute per IP
- Failed auth attempts logged and monitored

## Coding Standards

### TypeScript
- Strict null checks enabled
- No any types allowed
- Define interfaces for all DTOs

### Error Handling
```typescript
// Always use this pattern for payment errors:
try {
  const result = await processPayment(request);
  return result;
} catch (error) {
  logger.error('Payment failed', {
    transactionId: request.id,
    error: error.message,
    // Never log: card numbers, CVV, passwords
  });
  throw new PaymentError({
    code: 'PAYMENT_FAILED',
    message: 'Payment could not be processed',
    statusCode: 402
  });
}
```

## Testing
- Mock external payment gateway
- Never use real card numbers in tests
- Use test card numbers: 4111111111111111

## Database
- Use parameterized queries ONLY
- Enable query logging (sanitized)
- Transactions required for payment operations

## Compliance
- SOX: All financial transactions must be auditable
- PCI-DSS Level 1 certified
- Annual security audit required
```

### Example 2: Enterprise-Level Standards

`/Library/Application Support/ClaudeCode/CLAUDE.md`:
```markdown
# Bank IT - Enterprise Coding Standards

## Mandatory Security Requirements

ALL projects must comply with:

### Authentication
- Multi-factor authentication required for production access
- Passwords must meet complexity requirements (12+ chars, mixed case, numbers, symbols)
- Session timeout: 15 minutes of inactivity
- Failed login attempts: Max 5, then lockout

### Logging & Monitoring
- All authentication attempts (success/failure) must be logged
- All database modifications must be logged
- Logs must include: timestamp, user, action, IP address
- NEVER log: passwords, tokens, SSNs, credit cards, account numbers

### Data Protection
- Encrypt all PII (Personally Identifiable Information)
- Data retention: Follow bank policy (7 years for financial records)
- Right to deletion: GDPR compliance
- Data classification: Public, Internal, Confidential, Restricted

### Code Quality
- Code review required: Minimum 2 approvals
- Static analysis: SonarQube scan must pass
- Dependency scanning: No high/critical vulnerabilities
- License compliance: Only approved open-source licenses

### Deployment
- Production deployments: Change management approval required
- Rollback plan mandatory
- Deployment window: Outside business hours
- Automated testing in staging required

## Prohibited Practices

NEVER:
- Commit secrets/credentials to git
- Use production data in development
- Disable security features
- Skip code reviews for "quick fixes"
- Deploy directly to production
- Use unapproved third-party libraries
```

### Example 3: User Personal Memory

`~/.claude/CLAUDE.md`:
```markdown
# My Personal Preferences

## Code Style
- I prefer functional programming style
- Use const by default, let only when necessary
- Prefer array methods (map, filter, reduce) over loops

## Comments
- Add JSDoc comments to all public functions
- Explain the "why", not the "what"

## Git
- Commit message format: type(scope): description
- Types I use: feat, fix, refactor, test, docs

## My Common Patterns

### API Response Format I Prefer
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
}
```

### My Error Handling Pattern
```typescript
async function withErrorHandling<T>(
  fn: () => Promise<T>
): Promise<ApiResponse<T>> {
  try {
    const data = await fn();
    return { success: true, data };
  } catch (error) {
    return {
      success: false,
      error: {
        code: error.code || 'UNKNOWN_ERROR',
        message: error.message
      }
    };
  }
}
```
```

---

## Best Practices

### 1. Start Simple, Expand Over Time

**Day 1:**
```markdown
# Project Name
- Use TypeScript
- Test before commit
```

**After 1 month:**
```markdown
# Project Name

## Coding Standards
- TypeScript with strict mode
- ESLint + Prettier
- 80% test coverage

## Architecture
[Detailed architecture notes]

## Security
[Security requirements]
```

### 2. Keep Memory Focused

**Good (focused):**
```markdown
## Database Queries
- Always use parameterized queries
- Enable query logging
- Use transactions for multi-step operations
```

**Bad (too general):**
```markdown
## Database
Use the database properly and follow best practices for optimal performance and security.
```

### 3. Use Sections for Organization

```markdown
# Project Name

## [Section 1: Coding Standards]
...

## [Section 2: Architecture]
...

## [Section 3: Security]
...
```

### 4. Update Memory as Project Evolves

```bash
# Review memory quarterly
> /memory

# Add new patterns discovered
# Remove outdated conventions
# Update security requirements
```

### 5. Team Collaboration on Memory

```bash
# Create project memory collaboratively
git checkout -b feature/update-claude-memory
# Edit ./CLAUDE.md
git add ./CLAUDE.md
git commit -m "docs: update Claude Code memory with new security requirements"
git push
# Create PR for team review
```

### 6. Version Control Memory Files

```markdown
# ./CLAUDE.md header:
<!--
Version: 2.1.0
Last Updated: 2025-10-19
Owners: DevOps Team
Review Schedule: Quarterly
-->

# Banking API Standards
...
```

---

## Common Patterns

### Pattern 1: Conditional Memory

```markdown
## Code Style

When working in `/frontend`:
- Use React functional components
- Hooks over class components
- CSS Modules for styling

When working in `/backend`:
- Use Express middleware pattern
- Async/await for all async operations
- Joi for input validation
```

### Pattern 2: Role-Based Memory

```markdown
## For Backend Developers
- Focus on API performance
- Optimize database queries
- Implement caching strategies

## For Frontend Developers
- Focus on user experience
- Optimize bundle size
- Ensure accessibility (WCAG 2.1 AA)

## For DevOps
- Infrastructure as Code (Terraform)
- CI/CD pipeline maintenance
- Security scanning integration
```

### Pattern 3: Technology-Specific Memory

```markdown
## TypeScript Conventions
- Use strict mode
- Prefer interfaces over types
- Explicit return types for public functions

## Database (PostgreSQL)
- Use snake_case for column names
- Always use indexes on foreign keys
- Use EXPLAIN ANALYZE for slow queries

## Testing (Jest)
- Use describe/it blocks
- Mock external dependencies
- Snapshot tests for UI components
```

---

## Summary

In this section, you learned:

### Core Concepts
- Memory persists across sessions via CLAUDE.md files
- Three-level hierarchy: enterprise, project, user
- Higher levels override lower levels

### Implementation
- Creating and structuring CLAUDE.md files
- Writing specific, actionable memory
- Using examples and checklists
- Managing memory with commands

### Banking Applications
- Compliance-focused memory (PCI-DSS, SOX)
- Security requirements enforcement
- Team standards documentation
- Role-specific guidelines

---

## Next Steps

1. **[Continue to Section 7: Slash Commands](./07-slash-commands.md)** - Create custom commands
2. **[Review Memory Documentation](https://docs.claude.com/en/docs/claude-code/memory)** - Official memory guide
3. **Create your first CLAUDE.md** - Start with basic coding standards

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-5

**Official References:**
- **Memory Documentation**: https://docs.claude.com/en/docs/claude-code/memory
- **Best Practices**: https://docs.claude.com/en/docs/claude-code/best-practices
