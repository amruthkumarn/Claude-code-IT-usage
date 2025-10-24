# Phase 1.3.7: Agents & Sub-agents

**Learning Objectives:**
- Understand what agents are and how they specialize
- Learn when to use agents vs main Claude
- Define custom agents for specific tasks
- Configure agent tools and restrictions
- Implement banking data engineering agents

**Time Commitment:** 60 minutes

**Prerequisites:** Phase 1.3.1-1.3.6 completed

---

## ⚡ Quick Start (5 minutes)

**Goal:** Create and use your first specialized agent.

### Try This Right Now

```bash
# 1. Create agents directory
mkdir -p .claude/agents
cd .claude/agents

# 2. Create a security auditor agent
cat > security-auditor.json << 'EOF'
{
  "name": "security-auditor",
  "description": "Banking security and compliance auditor",
  "instructions": "You are a banking security expert. Review code for:\n- PCI-DSS compliance\n- Hardcoded secrets\n- PII data exposure\n- SQL injection vulnerabilities\nProvide severity ratings and remediation steps.",
  "tools": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash"]
  }
}
EOF

# 3. Use it
claude
> Use the security-auditor agent to review this project for security issues
```

**What you'll see:**
- Claude launches the security-auditor agent
- Agent scans files with read-only access
- Provides security findings with severity ratings

**Key Insight:** Agents are task-specific versions of Claude with focused expertise and restricted permissions.

---

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

### Example 6: Data Quality Validator

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

## 🔨 Exercise 1: Create Your Banking Agent Team (25 minutes)

**Goal:** Build a complete set of specialized agents for banking data engineering.

### Step 1: Set up agents configuration

```bash
cd ~/banking-pipeline-project
mkdir -p .claude/agents
```

### Step 2: Create .claude/agents.json

```bash
cat > .claude/agents.json << 'EOF'
{
  "agents": {
    "compliance-checker": {
      "description": "Banking compliance auditor (PCI-DSS, SOX, GDPR)",
      "prompt": "You are a banking compliance expert. Review code for:\n- PCI-DSS: Credit card data exposure, CVV storage\n- SOX: Audit trails, transaction logging\n- GDPR: PII handling, data retention\n\nProvide severity ratings and remediation steps with code examples.",
      "tools": ["Read", "Grep", "Glob"]
    },

    "security-auditor": {
      "description": "Security vulnerability scanner",
      "prompt": "You are a security expert. Scan for:\n- Hardcoded secrets and credentials\n- SQL injection vulnerabilities\n- Insecure data access patterns\n- Missing encryption\n- PII exposure in logs\n\nProvide file:line references and secure code examples.",
      "tools": ["Read", "Grep", "Glob"]
    },

    "test-generator": {
      "description": "PySpark test specialist",
      "prompt": "You are a testing expert. Generate comprehensive pytest tests for PySpark pipelines.\n\nInclude:\n- Happy path tests with sample DataFrames\n- Edge cases (nulls, empty data, duplicates)\n- Banking scenarios (overdrafts, invalid amounts)\n- Use pytest fixtures and chispa for DataFrame assertions\n\nFollow Arrange-Act-Assert pattern.",
      "tools": ["Read", "Write", "Grep"]
    },

    "doc-generator": {
      "description": "Pipeline documentation specialist",
      "prompt": "You generate comprehensive Markdown documentation for data pipelines.\n\nInclude:\n- Pipeline purpose and business logic\n- Input/output schemas\n- Transformation steps\n- Data quality checks\n- Banking compliance notes\n- Example queries\n\nFormat in clear Markdown with code blocks.",
      "tools": ["Read", "Write", "Grep", "Glob"]
    },

    "performance-optimizer": {
      "description": "PySpark performance expert",
      "prompt": "You are a PySpark optimization specialist. Analyze for:\n- Inefficient joins and aggregations\n- Missing caching and partitioning\n- Excessive shuffles\n- Small files in Delta Lake\n- Column pruning opportunities\n\nProvide optimized PySpark code with performance estimates.",
      "tools": ["Read", "Grep", "Glob"]
    }
  }
}
EOF
```

### Step 3: Add to settings.json

```bash
# Edit .claude/settings.json to reference agents.json
cat >> .claude/settings.json << 'EOF'
  "agentsFile": ".claude/agents.json"
EOF
```

### Step 4: Test each agent

```bash
claude

# Test compliance checker
> Use compliance-checker agent to audit this project for PCI-DSS compliance

# Test security auditor
> Use security-auditor agent to scan for hardcoded secrets

# Test test generator
> Use test-generator agent to create tests for pipelines/transaction_validator.py

# Test doc generator
> Use doc-generator agent to document the payment processing pipeline

# Test performance optimizer
> Use performance-optimizer agent to analyze pipelines/reporting/monthly_summary.py
```

### Step 5: Verify agent isolation

**Test 1: Verify read-only agents can't modify code**
```
> Use compliance-checker agent to fix the compliance issues it found
```

**Expected:** Agent will say it cannot modify files (only has Read, Grep, Glob)

**Test 2: Verify test-generator can write**
```
> Use test-generator agent to create tests for validators/amount_validator.py
```

**Expected:** Agent creates test file successfully (has Write permission)

### ✅ Checkpoint
- [ ] Created agents.json with 5 specialized agents
- [ ] Configured proper tool restrictions for each agent
- [ ] Tested each agent with appropriate tasks
- [ ] Verified read-only agents cannot modify code
- [ ] Verified write-enabled agents can create files

### 💻 Terminal Session Example

```bash
$ claude

> Use compliance-checker agent to audit pipelines/payment_processor.py

🤖 Launching compliance-checker agent...

═══════════════════════════════════════════════════
 COMPLIANCE CHECKER AGENT
═══════════════════════════════════════════════════

Analyzing: pipelines/payment_processor.py

🔧 Tool Use: Read
File: pipelines/payment_processor.py
[Reading file...]

🔧 Tool Use: Grep
Pattern: "card_number|cvv|password"
[Searching for sensitive data...]

📋 COMPLIANCE AUDIT REPORT

## PCI-DSS Findings

### CRITICAL: CVV Storage Detected
**File:** pipelines/payment_processor.py:42
**Issue:** CVV is being stored in transaction_data DataFrame
```python
# VIOLATION:
transaction_df.withColumn("cvv", F.col("card_cvv"))
```

**Regulation:** PCI-DSS Requirement 3.2
**Remediation:**
```python
# CORRECT: Never store CVV
transaction_df = transaction_df.drop("card_cvv")  # Drop before persisting
```

### HIGH: Insufficient PII Masking
**File:** pipelines/payment_processor.py:87
**Issue:** Account numbers logged without masking
```python
# VIOLATION:
logger.info(f"Processing account {account_number}")
```

**Regulation:** PCI-DSS Requirement 3.3
**Remediation:**
```python
# CORRECT: Mask account number
def mask_account(account: str) -> str:
    return f"****{account[-4:]}"

logger.info(f"Processing account {mask_account(account_number)}")
```

## SOX Findings

### MEDIUM: Missing Audit Trail
**File:** pipelines/payment_processor.py:120
**Issue:** Transaction updates lack audit logging
**Regulation:** SOX Section 404
**Remediation:** Add audit log table with user, timestamp, before/after values

═══════════════════════════════════════════════════
 SUMMARY
═══════════════════════════════════════════════════

Critical: 1
High: 1
Medium: 1
Low: 0

Agent completed successfully ✓
```

### 🎯 Challenge: Create More Agents

Try creating these additional agents:
1. **data-profiler**: Analyzes data distributions and quality
2. **schema-validator**: Validates DataFrame schemas match specifications
3. **cost-optimizer**: Suggests ways to reduce Spark cluster costs

<details>
<summary>💡 Example: data-profiler agent</summary>

```json
{
  "data-profiler": {
    "description": "Data profiling and statistics specialist",
    "prompt": "You analyze data quality and generate profiling reports.\n\nGenerate:\n- Row counts and null percentages\n- Value distributions for key columns\n- Duplicate detection\n- Data type mismatches\n- Outlier detection for numeric columns\n- Cardinality analysis\n\nFor banking data, focus on:\n- Transaction amount distributions\n- Account balance ranges\n- Date coverage gaps\n- Currency code frequencies",
    "tools": ["Read", "Grep", "Glob"]
  }
}
```

</details>

**Key Insight:** Agents enable you to build a "team of specialists" where each agent has focused expertise and appropriate permissions - just like a real data engineering team!

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
    "performance-optimizer": { "...": "..." },
    "data-quality-validator": { "...": "..." }
  }
}
```

Commit to git so entire team can use!

---

## Summary

In this subsection, you learned:

### Core Concepts
- ✅ Agents are specialized versions of Claude
- ✅ Each has its own context, tools, and model
- ✅ Useful for focused, repeatable tasks

### Implementation
- ✅ Defining agents via CLI or config files
- ✅ Tool restrictions for safety
- ✅ Model selection for cost/performance
- ✅ Using agents with @ syntax

### Banking Data Engineering Applications
- ✅ Compliance checking for data pipelines
- ✅ SQL and data warehouse security auditing
- ✅ Pipeline documentation generation
- ✅ Test coverage analysis for PySpark code
- ✅ Performance optimization for big data workloads
- ✅ Data quality validation and monitoring

---

## Next Steps

👉 **[Continue to 1.3.8: Hooks & Automation](./03-08-hooks-automation.md)**

**Quick Practice:**
1. Create a simple `@code-reviewer` agent
2. Use it to review a PySpark file
3. Create a `@test-generator` agent for pytest

---

**Related Sections:**
- [Phase 1.3.6: Slash Commands](./03-06-slash-commands.md) - Custom commands
- [Phase 1.3.8: Hooks & Automation](./03-08-hooks-automation.md) - Automation
- [Phase 1.3.11: Standards & Best Practices](./03-11-standards-best-practices.md) - Standards

---

**Last Updated:** 2025-10-24
**Version:** 1.0
