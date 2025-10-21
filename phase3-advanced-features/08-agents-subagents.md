# Section 8: Agents & Sub-agents

## Table of Contents
1. [Understanding Agents](#understanding-agents)
2. [When to Use Agents](#when-to-use-agents)
3. [Defining Custom Agents](#defining-custom-agents)
4. [Agent Configuration](#agent-configuration)
5. [Banking IT Agent Examples](#banking-it-agent-examples)
6. [Best Practices](#best-practices)

---

## Understanding Agents

**Agents** (or sub-agents) are specialized versions of Claude that focus on specific tasks with their own:
- Custom instructions/prompts
- Tool restrictions
- Model selection
- Context isolation

### Main Claude vs Sub-Agents

```
┌─────────────────────────────────────┐
│  Main Claude Session                │
│  - General purpose                  │
│  - Full context                     │
│  - All tools available              │
└──────────────┬──────────────────────┘
               │
               │ Delegates to:
               │
     ┌─────────┴─────────────────┬──────────────────────┐
     │                            │                       │
┌────▼──────────┐    ┌────────────▼──────┐    ┌────────▼──────────┐
│ Code Reviewer  │    │ Test Generator    │    │ Security Auditor  │
│ - Review focus │    │ - Test creation   │    │ - Security only   │
│ - Read-only    │    │ - Generate code   │    │ - Read-only       │
│ - Opus model   │    │ - Sonnet model    │    │ - Opus model      │
└────────────────┘    └───────────────────┘    └───────────────────┘
```

### Benefits of Agents

1. **Specialized Expertise** - Focused instructions for specific tasks
2. **Isolated Context** - Each agent has its own context window
3. **Tool Restrictions** - Limit what each agent can do
4. **Model Selection** - Use appropriate model per task
5. **Reusability** - Define once, use repeatedly

---

## When to Use Agents

### Good Use Cases

**Security Review:**
```bash
> @security-auditor review the payment processing code
```

**Test Generation:**
```bash
> @test-generator create tests for src/auth/login.ts
```

**Documentation:**
```bash
> @doc-writer generate API documentation for this endpoint
```

### When NOT to Use Agents

- Simple, one-off tasks
- When context sharing is critical
- When you need iterative back-and-forth
- For tasks requiring full system access

---

## Defining Custom Agents

### Method 1: Command Line

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer",
    "prompt": "You are a senior code reviewer. Focus on security, performance, and best practices.",
    "tools": ["Read", "Grep", "Glob"],
    "model": "opus"
  }
}'
```

### Method 2: Configuration File

`.claude/settings.json`:
```json
{
  "agents": {
    "code-reviewer": {
      "description": "Expert code reviewer",
      "prompt": "You are a senior code reviewer. Focus on security, performance, and maintainability.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "test-generator": {
      "description": "Test creation specialist",
      "prompt": "You are a testing expert. Generate comprehensive, maintainable tests.",
      "tools": ["Read", "Write", "Grep"],
      "model": "sonnet"
    }
  }
}
```

### Using Agents

```bash
# In Claude Code session:
> @code-reviewer review src/payments/processor.ts

> @test-generator create tests for src/auth/middleware.ts
```

---

## Agent Configuration

### Configuration Properties

```json
{
  "agent-name": {
    "description": "Short description (shown in /help)",
    "prompt": "Detailed instructions for the agent",
    "tools": ["Tool1", "Tool2"],  // Optional: Restrict tools
    "model": "sonnet|opus|haiku"  // Optional: Specify model
  }
}
```

### Available Tools for Restriction

```json
{
  "tools": [
    "Read",          // Read files
    "Write",         // Create files
    "Edit",          // Modify files
    "Grep",          // Search content
    "Glob",          // Find files
    "Bash",          // Execute commands
    "WebFetch",      // Fetch URLs
    "WebSearch",     // Search web
    "Task",          // Launch sub-agents
    "TodoWrite"      // Manage todos
  ]
}
```

### Example: Read-Only Agent

```json
{
  "security-auditor": {
    "description": "Security vulnerability scanner",
    "prompt": "You are a security expert. Identify vulnerabilities, never modify code.",
    "tools": ["Read", "Grep", "Glob"],
    "model": "opus"
  }
}
```

### Example: Full Access Agent

```json
{
  "feature-builder": {
    "description": "Complete feature implementation",
    "prompt": "You implement complete features including code, tests, and documentation.",
    "tools": ["Read", "Write", "Edit", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}
```

---

## Banking IT Agent Examples

### Example 1: Compliance Checker

```json
{
  "compliance-checker": {
    "description": "Banking compliance auditor (PCI-DSS, SOX, GDPR)",
    "prompt": "You are a banking compliance expert specializing in PCI-DSS, SOX, and GDPR regulations.\n\nWhen reviewing code, check for:\n\n## PCI-DSS\n- Credit card number exposure in logs\n- CVV storage (must NEVER be stored)\n- Encryption of cardholder data\n- Tokenization implementation\n- Access control to payment data\n\n## SOX\n- Audit trail for financial transactions\n- Separation of duties in code\n- Change management compliance\n- Database transaction integrity\n\n## GDPR\n- Personal data classification\n- Consent tracking\n- Right to deletion implementation\n- Data retention policies\n- Cross-border data transfer\n\nProvide:\n1. Severity (Critical/High/Medium/Low)\n2. Specific file and line numbers\n3. Regulation violated\n4. Remediation steps\n5. Code examples of fixes",
    "tools": ["Read", "Grep", "Glob"],
    "model": "opus"
  }
}
```

**Usage:**
```bash
> @compliance-checker audit the payment processing module for PCI-DSS compliance
```

### Example 2: SQL Security Auditor

```json
{
  "sql-auditor": {
    "description": "Database security specialist",
    "prompt": "You are a database security expert. Analyze SQL queries and database interactions for:\n\n1. SQL Injection vulnerabilities\n2. Missing parameterized queries\n3. Exposed sensitive data in queries\n4. Missing indexes on foreign keys\n5. N+1 query problems\n6. Missing transactions for multi-step operations\n7. Improper error handling exposing DB structure\n\nFor each issue found:\n- Show vulnerable code\n- Explain the risk\n- Provide secure alternative\n- Rate severity (Critical/High/Medium/Low)\n\nFocus on banking-critical data: accounts, transactions, customer PII.",
    "tools": ["Read", "Grep"],
    "model": "opus"
  }
}
```

**Usage:**
```bash
> @sql-auditor review all database queries in src/repositories/
```

### Example 3: API Documentation Generator

```json
{
  "api-doc-generator": {
    "description": "OpenAPI/Swagger documentation specialist",
    "prompt": "You generate comprehensive OpenAPI 3.0 documentation for REST APIs.\n\nInclude:\n- All endpoints with methods (GET, POST, PUT, DELETE, PATCH)\n- Request/response schemas with examples\n- Authentication requirements (JWT, OAuth2)\n- Error responses (4xx, 5xx)\n- Rate limiting information\n- Pagination details\n\nBanking-specific requirements:\n- Document validation rules\n- Include audit logging notes\n- Specify transaction boundaries\n- Note idempotency requirements\n- Document retry policies\n\nGenerate valid OpenAPI YAML format.",
    "tools": ["Read", "Grep", "Write"],
    "model": "sonnet"
  }
}
```

**Usage:**
```bash
> @api-doc-generator create OpenAPI docs for src/api/payments/
```

### Example 4: Test Coverage Analyzer

```json
{
  "test-analyzer": {
    "description": "Test coverage and quality analyzer",
    "prompt": "You are a testing expert. Analyze test coverage and quality.\n\nFor the codebase, identify:\n1. Functions/methods without tests\n2. Test coverage percentage\n3. Missing edge cases in existing tests\n4. Missing error condition tests\n5. Inadequate mocking\n\nBanking-specific test scenarios:\n- Insufficient funds\n- Account overdraft limits\n- Transaction rollback scenarios\n- Concurrent transaction handling\n- Regulatory compliance validation\n\nProvide:\n- Coverage report\n- List of untested code paths\n- Generated test cases for gaps\n- Prioritization (critical paths first)",
    "tools": ["Read", "Grep", "Glob"],
    "model": "sonnet"
  }
}
```

**Usage:**
```bash
> @test-analyzer analyze test coverage for src/transactions/
```

### Example 5: Performance Optimizer

```json
{
  "performance-optimizer": {
    "description": "Performance analysis and optimization specialist",
    "prompt": "You are a performance optimization expert. Analyze code for:\n\n1. Algorithmic inefficiencies (O(n²) → O(n log n))\n2. Database query optimization\n3. Unnecessary API calls\n4. Memory leaks\n5. Blocking operations that should be async\n6. Missing caching opportunities\n7. Unindexed database queries\n\nBanking context:\n- High transaction volume handling\n- Real-time balance calculations\n- Report generation optimization\n- Batch processing efficiency\n\nFor each issue:\n- Current performance profile\n- Bottleneck explanation\n- Optimized solution\n- Expected performance improvement\n- Trade-offs to consider",
    "tools": ["Read", "Grep", "Glob"],
    "model": "opus"
  }
}
```

### Example 6: Migration Generator

```json
{
  "migration-generator": {
    "description": "Database migration specialist",
    "prompt": "You generate safe, reversible database migrations for PostgreSQL.\n\nRequirements:\n- Wrapped in transactions\n- Include UP and DOWN migrations\n- Add indexes for foreign keys\n- Use banking naming conventions (snake_case)\n- Add descriptive comments\n- Consider data migration if schema changes\n\nInclude:\n- Rollback strategy\n- Data type justification\n- Index rationale\n- Foreign key constraints\n- NOT NULL constraints with defaults\n\nValidate:\n- No data loss\n- Backward compatibility\n- Performance impact assessment",
    "tools": ["Read", "Write", "Grep"],
    "model": "sonnet"
  }
}
```

**Usage:**
```bash
> @migration-generator create migration to add transaction_audit table
```

---

## Best Practices

### 1. Focused Prompts

**Good:**
```json
{
  "security-reviewer": {
    "prompt": "You are a security expert. Identify:\n1. SQL injection\n2. XSS vulnerabilities\n3. Authentication bypass\n4. Sensitive data exposure\n\nProvide specific file:line references and fixes."
  }
}
```

**Bad:**
```json
{
  "security-reviewer": {
    "prompt": "Review code for security"
  }
}
```

### 2. Appropriate Tool Restrictions

**Read-Only Agents:**
```json
{
  "auditor": {
    "tools": ["Read", "Grep", "Glob"]
  }
}
```

**Code Generation Agents:**
```json
{
  "generator": {
    "tools": ["Read", "Write", "Grep", "Glob"]
  }
}
```

### 3. Right Model for the Task

```json
{
  "simple-formatter": {
    "model": "haiku"  // Fast, cheap
  },
  "architect": {
    "model": "opus"   // Complex decisions
  },
  "general-dev": {
    "model": "sonnet" // Balanced
  }
}
```

### 4. Clear Descriptions

```json
{
  "compliance-checker": {
    "description": "Banking compliance auditor (PCI-DSS, SOX, GDPR)"
  }
}
```

Shows up in `/help` output.

### 5. Domain-Specific Context

```json
{
  "payment-reviewer": {
    "prompt": "You review payment processing code.\n\nContext:\n- We use Stripe API\n- All amounts in cents (integer)\n- USD currency only\n- Idempotency keys required\n- Webhook signature validation mandatory\n\nCheck for these common issues:\n- Floating point for money (should be integer)\n- Missing idempotency\n- Unverified webhooks\n- Missing retry logic"
  }
}
```

### 6. Banking Agent Collection

`.claude/settings.json`:
```json
{
  "agents": {
    "compliance-checker": { "...": "..." },
    "sql-auditor": { "...": "..." },
    "security-reviewer": { "...": "..." },
    "test-analyzer": { "...": "..." },
    "api-doc-generator": { "...": "..." },
    "migration-generator": { "...": "..." },
    "performance-optimizer": { "...": "..." }
  }
}
```

Commit to git so entire team can use!

---

## Summary

In this section, you learned:

### Core Concepts
- Agents are specialized versions of Claude
- Each has its own context, tools, and model
- Useful for focused, repeatable tasks

### Implementation
- Defining agents via CLI or config files
- Tool restrictions for safety
- Model selection for cost/performance
- Using agents with @ syntax

### Banking Applications
- Compliance checking agents
- SQL security auditing
- API documentation generation
- Test coverage analysis
- Performance optimization
- Database migration generation

---

## Next Steps

1. **[Continue to Section 9: Security & Compliance](../phase4-enterprise-security/09-security-compliance.md)** - Deep dive into security
2. **[Review Agent Documentation](https://docs.claude.com/en/docs/claude-code/agents)** - Official agent guide
3. **Create your first agent** - Start with a simple code-reviewer

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-7

**Official References:**
- **Agents Guide**: https://docs.claude.com/en/docs/claude-code/agents
- **Sub-agents Documentation**: https://docs.claude.com/en/docs/claude-code/subagents
