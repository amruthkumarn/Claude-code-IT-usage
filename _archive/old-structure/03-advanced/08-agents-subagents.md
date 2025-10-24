# Section 8: Agents & Sub-agents

## Table of Contents
1. [Understanding Agents](#understanding-agents)
2. [When to Use Agents](#when-to-use-agents)
3. [Defining Custom Agents](#defining-custom-agents)
4. [Agent Configuration](#agent-configuration)
5. [Banking Data Engineering Agent Examples](#banking-data-engineering-agent-examples)
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
│ Code Reviewer  │    │ Pipeline Tester   │    │ Security Auditor  │
│ - Review focus │    │ - Test creation   │    │ - Security only   │
│ - Read-only    │    │ - Generate code   │    │ - Read-only       │
│ - Sonnet model │    │ - Sonnet model    │    │ - Sonnet model    │
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
> @security-auditor review the payment processing pipeline
```

**Pipeline Testing:**
```bash
> @pipeline-tester create tests for src/pipelines/transactions_etl.py
```

**Documentation:**
```bash
> @doc-writer generate documentation for this data pipeline
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
    "description": "Expert PySpark code reviewer",
    "prompt": "You are a senior PySpark data engineer. Focus on security, performance, and best practices.",
    "tools": ["Read", "Grep", "Glob"]
  }
}'
```

### Method 2: Configuration File

`.claude/settings.json`:
```json
{
  "agents": {
    "code-reviewer": {
      "description": "Expert PySpark code reviewer",
      "prompt": "You are a senior PySpark data engineer. Focus on security, performance, and maintainability.",
      "tools": ["Read", "Grep", "Glob"]
    },
    "pipeline-tester": {
      "description": "Data pipeline test specialist",
      "prompt": "You are a testing expert. Generate comprehensive, maintainable pytest tests for PySpark pipelines.",
      "tools": ["Read", "Write", "Grep"]
    }
  }
}
```

### Using Agents

```bash
# In Claude Code session:
> @code-reviewer review src/pipelines/payment_processor.py

> @pipeline-tester create tests for src/transforms/transaction_aggregator.py
```

---

## Agent Configuration

### Configuration Properties

```json
{
  "agent-name": {
    "description": "Short description (shown in /help)",
    "prompt": "Detailed instructions for the agent",
    "tools": ["Tool1", "Tool2"]  // Optional: Restrict tools
  }
}
```

**Note:** The `model` field is optional. When omitted, agents use the default model (Sonnet).

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
    "tools": ["Read", "Grep", "Glob"]
  }
}
```

### Example: Full Access Agent

```json
{
  "pipeline-builder": {
    "description": "Complete data pipeline implementation",
    "prompt": "You implement complete PySpark pipelines including code, tests, and documentation.",
    "tools": ["Read", "Write", "Edit", "Grep", "Glob", "Bash"]
  }
}
```

---

## Banking Data Engineering Agent Examples

### Example 1: Compliance Checker

```json
{
  "compliance-checker": {
    "description": "Banking data compliance auditor (PCI-DSS, SOX, GDPR)",
    "prompt": "You are a banking data compliance expert specializing in PCI-DSS, SOX, and GDPR regulations.\n\nWhen reviewing PySpark pipelines and Python code, check for:\n\n## PCI-DSS\n- Credit card number exposure in logs or DataFrames\n- CVV storage (must NEVER be stored or persisted)\n- Encryption of cardholder data at rest and in transit\n- Tokenization implementation in data pipelines\n- Access control to payment data in Delta/Parquet tables\n- Masking of sensitive columns in non-production environments\n\n## SOX\n- Audit trail for financial transaction processing\n- Separation of duties in pipeline orchestration\n- Change management compliance (version control)\n- Data lineage tracking\n- Immutable audit logs\n\n## GDPR\n- Personal data classification in data catalogs\n- Consent tracking in customer tables\n- Right to deletion implementation (soft deletes)\n- Data retention policies in pipeline configurations\n- Cross-border data transfer compliance\n- PII anonymization in analytical datasets\n\nProvide:\n1. Severity (Critical/High/Medium/Low)\n2. Specific file and line numbers\n3. Regulation violated\n4. Remediation steps with PySpark code examples\n5. Delta Lake/Iceberg recommendations for compliance",
    "tools": ["Read", "Grep", "Glob"]
  }
}
```

**Usage:**
```bash
> @compliance-checker audit the payment processing pipeline for PCI-DSS compliance
```

### Example 2: SQL Security Auditor

```json
{
  "sql-auditor": {
    "description": "Data warehouse security specialist",
    "prompt": "You are a database and data warehouse security expert. Analyze Spark SQL, Delta Lake operations, and database interactions for:\n\n1. SQL Injection vulnerabilities in dynamic queries\n2. Missing parameterized queries in JDBC/ODBC connections\n3. Exposed sensitive data in Spark SQL queries and DataFrames\n4. Missing Z-ORDER or partition optimization on critical columns\n5. Inefficient join patterns (broadcast vs sort-merge)\n6. Missing ACID transactions for multi-step Delta Lake operations\n7. Improper error handling exposing data warehouse structure\n8. Unencrypted connections to data sources\n9. Hardcoded credentials in connection strings\n\nFor each issue found:\n- Show vulnerable PySpark/SQL code\n- Explain the risk in banking context\n- Provide secure alternative with code example\n- Rate severity (Critical/High/Medium/Low)\n\nFocus on banking-critical data: accounts, transactions, customer PII, balances.",
    "tools": ["Read", "Grep"]
  }
}
```

**Usage:**
```bash
> @sql-auditor review all SQL queries and data access in src/repositories/
```

### Example 3: Pipeline Documentation Generator

```json
{
  "pipeline-doc-generator": {
    "description": "Data pipeline documentation specialist",
    "prompt": "You generate comprehensive documentation for PySpark data pipelines.\n\nInclude:\n- Pipeline purpose and business logic\n- Input sources (tables, files, APIs) with schemas\n- Output destinations (Delta Lake tables, S3 paths) with schemas\n- Transformation logic with example data\n- Data quality checks and validation rules\n- Performance characteristics (runtime, data volume)\n- Dependencies and upstream/downstream pipelines\n- Configuration parameters and environment variables\n- Error handling and retry logic\n- Monitoring and alerting setup\n\nBanking-specific requirements:\n- Document validation rules for financial data\n- Include audit logging notes\n- Specify transaction boundaries and ACID guarantees\n- Note idempotency requirements\n- Document data retention and archival policies\n- Include sample queries for business users\n\nGenerate Markdown format with code examples.",
    "tools": ["Read", "Grep", "Write"]
  }
}
```

**Usage:**
```bash
> @pipeline-doc-generator create documentation for src/pipelines/payments_etl/
```

### Example 4: Test Coverage Analyzer

```json
{
  "test-analyzer": {
    "description": "Pipeline test coverage and quality analyzer",
    "prompt": "You are a data engineering testing expert. Analyze test coverage and quality for PySpark pipelines.\n\nFor the codebase, identify:\n1. Functions/transformations without unit tests\n2. Test coverage percentage using pytest-cov\n3. Missing edge cases in existing tests\n4. Missing data quality validation tests\n5. Inadequate DataFrame mocking with chispa or spark-testing-base\n6. Missing integration tests for end-to-end pipelines\n\nBanking-specific test scenarios:\n- Invalid transaction amounts (negative, zero, exceeding limits)\n- Account balance constraints (overdraft, insufficient funds)\n- Transaction rollback and reconciliation scenarios\n- Concurrent transaction handling and race conditions\n- Duplicate transaction detection (idempotency)\n- Date boundary conditions (end of month, year transitions)\n- Currency conversion edge cases\n- Regulatory compliance validation\n\nProvide:\n- Coverage report from pytest-cov\n- List of untested transformation functions\n- Generated pytest test cases for gaps\n- PySpark DataFrame test fixtures using chispa\n- Prioritization (critical data quality paths first)",
    "tools": ["Read", "Grep", "Glob"]
  }
}
```

**Usage:**
```bash
> @test-analyzer analyze test coverage for src/pipelines/transactions/
```

### Example 5: Performance Optimizer

```json
{
  "performance-optimizer": {
    "description": "PySpark performance analysis and optimization specialist",
    "prompt": "You are a PySpark performance optimization expert. Analyze code for:\n\n1. Inefficient transformations (multiple passes over data)\n2. Missing partition pruning in Delta Lake queries\n3. Suboptimal join strategies (sort-merge vs broadcast)\n4. Excessive shuffle operations\n5. Missing caching for reused DataFrames\n6. Small files problem in Delta Lake tables\n7. Unoptimized column selection (SELECT * instead of specific columns)\n8. Missing predicate pushdown to data sources\n9. Unnecessary UDFs that could be built-in Spark functions\n10. Memory pressure from collect() or toPandas() operations\n\nBanking context:\n- High transaction volume processing (millions of records)\n- Real-time balance calculations with aggregations\n- Monthly/quarterly report generation optimization\n- Batch processing efficiency for EOD settlement\n- Streaming pipeline optimization (Structured Streaming)\n\nFor each issue:\n- Current performance profile (explain plan analysis)\n- Bottleneck explanation with Spark UI evidence\n- Optimized PySpark solution with code\n- Expected performance improvement estimate\n- Trade-offs to consider (memory vs speed)\n- Delta Lake optimization commands (OPTIMIZE, VACUUM, Z-ORDER)",
    "tools": ["Read", "Grep", "Glob"]
  }
}
```

**Usage:**
```bash
> @performance-optimizer analyze pipeline performance in src/pipelines/reporting/
```

### Example 6: Schema Migration Generator

```json
{
  "schema-migration-generator": {
    "description": "Delta Lake schema evolution specialist",
    "prompt": "You generate safe, backward-compatible schema migrations for Delta Lake tables in Python/PySpark.\n\nRequirements:\n- Use Delta Lake schema evolution features (mergeSchema, overwriteSchema)\n- Generate Alembic or custom migration scripts\n- Include rollback strategy\n- Use banking naming conventions (snake_case)\n- Add descriptive comments and docstrings\n- Consider data backfill if schema changes\n- Handle NULL constraints and default values\n\nInclude:\n- DDL for table creation/alteration\n- PySpark code for data migration\n- Data type justification (DecimalType for money)\n- Partition strategy (daily, monthly)\n- Z-ORDER optimization recommendations\n- Column comments for data catalog\n\nValidate:\n- No data loss during migration\n- Backward compatibility with existing pipelines\n- Performance impact assessment\n- Time travel capability preserved\n\nExample output format:\n```python\n# Migration: add_transaction_audit_columns\n# Date: 2025-10-23\n# Author: Data Engineering Team\n\nfrom pyspark.sql import SparkSession\nfrom pyspark.sql.types import *\nfrom delta.tables import DeltaTable\n\ndef upgrade(spark: SparkSession):\n    # Migration code here\n    pass\n\ndef downgrade(spark: SparkSession):\n    # Rollback code here\n    pass\n```",
    "tools": ["Read", "Write", "Grep"]
  }
}
```

**Usage:**
```bash
> @schema-migration-generator create migration to add audit columns to transactions table
```

### Example 7: Data Quality Validator

```json
{
  "data-quality-validator": {
    "description": "Data quality and validation specialist",
    "prompt": "You are a data quality expert for banking data pipelines. Generate comprehensive data quality checks using Great Expectations, Deequ, or custom PySpark validations.\n\nChecks to implement:\n\n## Completeness\n- NULL checks on critical columns (account_id, transaction_amount)\n- Required field presence\n- Referential integrity (foreign keys)\n\n## Accuracy\n- Transaction amount ranges (min, max)\n- Balance calculations (debits - credits = balance)\n- Currency code validation (ISO 4217)\n- Date ranges (transaction_date within valid period)\n\n## Consistency\n- Duplicate transaction detection\n- Account status transitions (active -> closed, not closed -> active)\n- Cross-field validation (debit_account != credit_account)\n\n## Timeliness\n- Data freshness checks (latest transaction within SLA)\n- Delayed transaction alerts\n\n## Banking-specific\n- Regulatory reporting thresholds (> $10,000 transactions)\n- Negative balance detection for non-overdraft accounts\n- Suspicious pattern detection (velocity checks)\n\nGenerate:\n- PySpark validation functions with proper error handling\n- Great Expectations expectations suite (JSON/YAML)\n- Data quality metrics collection\n- Alerting logic for quality violations\n- Quarantine table logic for bad records",
    "tools": ["Read", "Write", "Grep", "Glob"]
  }
}
```

**Usage:**
```bash
> @data-quality-validator create validation suite for transactions pipeline
```

---

## Best Practices

### 1. Focused Prompts

**Good:**
```json
{
  "security-reviewer": {
    "prompt": "You are a PySpark security expert. Identify:\n1. SQL injection in dynamic Spark SQL\n2. Hardcoded credentials in connection configs\n3. Unencrypted sensitive columns\n4. Missing access controls on Delta tables\n\nProvide specific file:line references and PySpark fixes."
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

**Pipeline Generation Agents:**
```json
{
  "generator": {
    "tools": ["Read", "Write", "Grep", "Glob", "Bash"]
  }
}
```

### 3. Use Default Model

```json
{
  "agent-name": {
    "description": "Agent description",
    "prompt": "Agent prompt",
    "tools": ["Read", "Grep"]
    // model field is optional - uses Sonnet by default
  }
}
```

**Note:** Agents use Sonnet by default, which is suitable for all data engineering tasks in AWS Bedrock environments.

### 4. Clear Descriptions

```json
{
  "compliance-checker": {
    "description": "Banking data compliance auditor (PCI-DSS, SOX, GDPR)"
  }
}
```

Shows up in `/help` output.

### 5. Domain-Specific Context

```json
{
  "payment-pipeline-reviewer": {
    "prompt": "You review payment processing PySpark pipelines.\n\nContext:\n- We use Delta Lake for all tables\n- Databricks runtime 14.3 LTS\n- All monetary amounts stored as Decimal(18,2)\n- USD currency only\n- Idempotency via upsert (merge) operations\n- Partitioned by transaction_date\n- Z-ORDERED by account_id for performance\n\nCheck for these common issues:\n- Float/Double for money (should be Decimal)\n- Missing idempotency keys in merge operations\n- Unpartitioned writes causing small files\n- Missing data quality checks\n- Missing audit logging"
  }
}
```

### 6. Banking Data Engineering Agent Collection

`.claude/settings.json`:
```json
{
  "agents": {
    "compliance-checker": { "...": "..." },
    "sql-auditor": { "...": "..." },
    "security-reviewer": { "...": "..." },
    "test-analyzer": { "...": "..." },
    "pipeline-doc-generator": { "...": "..." },
    "schema-migration-generator": { "...": "..." },
    "performance-optimizer": { "...": "..." },
    "data-quality-validator": { "...": "..." }
  }
}
```

Commit to git so entire team can use!

### 7. Environment-Specific Configuration

**Development Environment:**
```python
# config/dev_agents.py
DEV_AGENT_TOOLS = ["Read", "Write", "Edit", "Grep", "Glob", "Bash"]

dev_agents = {
    "pipeline-builder": {
        "description": "Full pipeline development agent",
        "tools": DEV_AGENT_TOOLS
    }
}
```

**Production Environment:**
```python
# config/prod_agents.py
PROD_AGENT_TOOLS = ["Read", "Grep", "Glob"]  # Read-only in production

prod_agents = {
    "pipeline-validator": {
        "description": "Production pipeline validator",
        "tools": PROD_AGENT_TOOLS
    }
}
```

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

### Banking Data Engineering Applications
- Compliance checking for data pipelines
- SQL and data warehouse security auditing
- Pipeline documentation generation
- Test coverage analysis for PySpark code
- Performance optimization for big data workloads
- Delta Lake schema evolution and migration
- Data quality validation and monitoring

---

## Next Steps

1. **[Continue to Section 9: Security & Compliance](../04-security/09-security-compliance.md)** - Deep dive into security
2. **[Review Agent Documentation](https://docs.claude.com/en/docs/claude-code/agents)** - Official agent guide
3. **Create your first agent** - Start with a simple pipeline-reviewer
4. **Setup Great Expectations** - Integrate data quality framework
5. **Configure Delta Lake optimizations** - Add OPTIMIZE and VACUUM agents

---

## Additional Resources

**Banking Data Engineering:**
- **PySpark Best Practices**: https://spark.apache.org/docs/latest/api/python/
- **Delta Lake Documentation**: https://docs.delta.io/latest/index.html
- **Great Expectations**: https://docs.greatexpectations.io/
- **Databricks Best Practices**: https://docs.databricks.com/

**Compliance & Security:**
- **PCI-DSS Data Security Standard**: https://www.pcisecuritystandards.org/
- **AWS Financial Services Compliance**: https://aws.amazon.com/financial-services/security-compliance/

**Official Claude References:**
- **Agents Guide**: https://docs.claude.com/en/docs/claude-code/agents
- **Sub-agents Documentation**: https://docs.claude.com/en/docs/claude-code/subagents
