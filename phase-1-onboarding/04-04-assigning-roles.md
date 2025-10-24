# Phase 1.4.4: Assigning Roles (Role Prompting)

**Learning Objectives:**
- Understand how role assignment improves output quality
- Learn to craft effective role definitions
- Apply role prompting to banking data engineering
- Combine roles with context for better results

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.3 completed

---

## Table of Contents
1. [What is Role Prompting?](#what-is-role-prompting)
2. [Why Roles Matter](#why-roles-matter)
3. [How to Assign Roles Effectively](#how-to-assign-roles-effectively)
4. [Banking Data Engineering Roles](#banking-data-engineering-roles)
5. [Roles in CLAUDE.md vs. Inline Prompts](#roles-in-claudemd-vs-inline-prompts)
6. [Practice Exercises](#practice-exercises)
7. [Summary](#summary)

---

## What is Role Prompting?

### Definition

**Role prompting** = Telling Claude what perspective, expertise, or persona to adopt when responding.

**Basic Structure:**
```
> You are a [ROLE].
> [TASK with context]
```

### Example Comparison

**Without Role:**
```
> Review this PySpark pipeline for issues
```

**With Role:**
```
> You are a senior banking data engineer with 10 years of PCI-DSS compliance experience.
>
> Review this PySpark transaction processing pipeline for:
> - Data security issues (PCI-DSS violations)
> - Performance bottlenecks
> - Code quality issues
```

**Result:** With the role, Claude focuses on banking-specific concerns and compliance requirements.

---

## Why Roles Matter

### Roles Shape Perspective

Different roles lead to different outputs:

#### Example: Code Review with Different Roles

**Code to Review:**
```python
def process_transactions(df: DataFrame) -> DataFrame:
    """Process daily transactions."""
    return df.filter(col("amount") > 0) \
             .groupBy("account_id") \
             .agg(sum("amount").alias("total"))
```

**Role 1: Junior Developer**
```
> You are a junior data engineer learning PySpark.
> Review this code and ask clarifying questions.
```

**Output:**
- "What does the filter do?"
- "Why use groupBy?"
- "What is agg()?"

**Role 2: Senior Data Engineer**
```
> You are a senior data engineer focused on production best practices.
> Review this code for improvements.
```

**Output:**
- "Add type hints for return type"
- "Add docstring with parameters and return value"
- "Add error handling for null amounts"
- "Consider partitioning strategy for large datasets"

**Role 3: Security Engineer**
```
> You are a banking security engineer ensuring PCI-DSS compliance.
> Review this code for security issues.
```

**Output:**
- "Are transactions containing PII being logged?"
- "Is account_id being masked in any outputs?"
- "Should we validate that sensitive fields (CVV, full card number) are not present?"
- "Add audit logging for transaction access"

**Role 4: Performance Engineer**
```
> You are a Spark performance optimization specialist.
> Review this code for performance improvements.
```

**Output:**
- "groupBy will trigger a shuffle - consider partition key optimization"
- "For large datasets, consider using partitionBy('account_id') on source data"
- "Add .cache() if this DataFrame is reused"
- "Consider broadcast join if enriching with small dimension tables"

### Roles Set Expectations

Roles help Claude understand:
- **Depth of explanation** (beginner vs. expert)
- **Focus areas** (security vs. performance vs. functionality)
- **Output format** (educational vs. production-ready)
- **Assumptions** (banking domain knowledge)

---

## How to Assign Roles Effectively

### 1. Be Specific About Expertise

**❌ Vague Role:**
```
> You are an engineer.
```

**✅ Specific Role:**
```
> You are a senior data engineer specializing in PySpark-based ETL pipelines
> for banking transaction processing, with expertise in PCI-DSS compliance.
```

### 2. Include Relevant Experience

**❌ Generic:**
```
> You are a data engineer.
```

**✅ With Experience:**
```
> You are a principal data engineer with 10 years of experience building
> real-time payment processing systems at major banks, specializing in
> fraud detection and regulatory compliance (PCI-DSS, SOX, GDPR).
```

### 3. Specify the Perspective

**❌ No Perspective:**
```
> Review this code.
```

**✅ With Perspective:**
```
> You are a code reviewer for a banking IT team.
> Your priority is ensuring production readiness, security compliance (PCI-DSS),
> and maintainability.
>
> Review this transaction validation pipeline and provide actionable feedback.
```

### 4. Combine Role with Task Context

**Complete Prompt Structure:**
```
┌─────────────────────────────────────┐
│ ROLE (Who are you?)                 │
│ You are a [expertise + experience]  │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ CONTEXT (What's the situation?)     │
│ Current state, constraints, goals   │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ TASK (What should you do?)          │
│ Specific action to perform          │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ FORMAT (How should output look?)    │
│ Expected output structure           │
└─────────────────────────────────────┘
```

**Example:**
```
> ROLE: You are a senior PySpark developer specializing in banking data pipelines
> with expertise in Delta Lake and streaming transactions.
>
> CONTEXT: We're building a real-time fraud detection system that processes
> credit card transactions. Current latency is 500ms per transaction.
> Target is <100ms. Cluster: 8 executors, 16GB each.
>
> TASK: Design a PySpark Structured Streaming pipeline to:
> 1. Ingest transactions from Kafka topic 'transactions'
> 2. Join with customer risk profiles (Delta table, 1M rows)
> 3. Calculate fraud score using windowed aggregations
> 4. Write high-risk transactions to 'fraud-alerts' topic
>
> FORMAT: Provide PySpark code with:
> - Complete streaming query definition
> - Window specifications for velocity checks
> - Broadcast join for risk profiles
> - Checkpointing configuration
> - Monitoring metrics
```

---

## Banking Data Engineering Roles

### Common Roles for Banking IT

**1. Senior Data Engineer**
```
> You are a senior data engineer at a major bank with 10+ years of experience
> building PySpark-based ETL pipelines for transaction processing, customer data,
> and regulatory reporting.
```

**Use for:**
- Pipeline design
- Architecture decisions
- Best practices guidance

---

**2. PCI-DSS Compliance Officer**
```
> You are a PCI-DSS compliance officer responsible for ensuring all data pipelines
> meet Payment Card Industry Data Security Standards. You review code for potential
> security violations, data exposure risks, and audit trail requirements.
```

**Use for:**
- Security reviews
- Compliance checks
- Sensitive data handling

---

**3. Performance Optimization Specialist**
```
> You are a Spark performance tuning specialist with expertise in optimizing
> large-scale data pipelines processing billions of banking transactions.
> You focus on shuffle optimization, partition strategies, and resource utilization.
```

**Use for:**
- Performance tuning
- Cluster optimization
- Scalability improvements

---

**4. Data Quality Engineer**
```
> You are a data quality engineer specializing in financial data validation.
> You ensure transaction data meets business rules, regulatory requirements,
> and data integrity standards.
```

**Use for:**
- Validation logic
- Data quality checks
- Reconciliation processes

---

**5. Test Engineer**
```
> You are a test automation engineer specializing in PySpark pipeline testing.
> You write comprehensive pytest test suites covering unit tests, integration tests,
> and data quality tests for banking data pipelines.
```

**Use for:**
- Test case generation
- Test data creation
- Test coverage analysis

---

**6. Code Reviewer**
```
> You are a technical lead conducting code reviews for a banking data engineering team.
> You ensure code quality, security compliance, maintainability, and adherence to
> team standards (PEP 8, type hints, docstrings).
```

**Use for:**
- Code reviews
- Refactoring suggestions
- Standards enforcement

---

**7. Database Architect**
```
> You are a data architect specializing in Delta Lake and data lakehouse design
> for banking applications. You design schemas, partitioning strategies, and
> data organization patterns for optimal query performance and regulatory compliance.
```

**Use for:**
- Schema design
- Partitioning strategies
- Data modeling

---

**8. DevOps Engineer**
```
> You are a DevOps engineer managing CI/CD pipelines for PySpark applications
> in a banking environment. You focus on automated testing, deployment strategies,
> and production monitoring.
```

**Use for:**
- CI/CD pipeline setup
- Deployment automation
- Monitoring configuration

---

## Roles in CLAUDE.md vs. Inline Prompts

### Two Places to Assign Roles

#### 1. In CLAUDE.md (Project-Level Role)

**Use for: Consistent role across all conversations**

`.claude/CLAUDE.md`:
```markdown
# Payment Processing Pipeline

## Your Role
You are a senior data engineer on the Payment Processing team at XYZ Bank.

**Your expertise:**
- PySpark 3.5+ for large-scale transaction processing
- Delta Lake for ACID transactions and time travel
- PCI-DSS compliance for payment card data
- SOX compliance for financial reporting
- Real-time streaming with Kafka

**Your responsibilities:**
- Design and implement transaction processing pipelines
- Ensure data security and regulatory compliance
- Optimize pipeline performance for 10M+ daily transactions
- Maintain data quality and audit trails
- Review code for production readiness

**Your standards:**
- All code must include type hints and comprehensive docstrings
- All PII must be masked in logs and non-production environments
- All pipelines must have ≥80% test coverage
- All changes must pass PCI-DSS compliance checks
```

**Effect:** Every prompt in this project will be answered from this perspective.

#### 2. In Inline Prompts (Task-Specific Role)

**Use for: Different perspective for a specific task**

**Example:**
```
> For this task, act as a security auditor (not your usual data engineer role).
>
> Audit the transaction_loader.py pipeline for PCI-DSS violations:
> - Check for PAN (Primary Account Number) exposure in logs
> - Verify CVV is never stored
> - Confirm sensitive data is encrypted at rest
> - Review audit trail completeness
```

**Effect:** Temporarily overrides the CLAUDE.md role for this specific task.

### When to Use Each

| Scenario | Use CLAUDE.md Role | Use Inline Role |
|----------|-------------------|-----------------|
| Day-to-day development | ✅ | ❌ |
| Consistent perspective needed | ✅ | ❌ |
| Special one-time task | ❌ | ✅ |
| Different expertise needed | ❌ | ✅ |
| Onboarding new team member | ✅ | ❌ |
| Security audit | ❌ | ✅ |
| Performance review | ❌ | ✅ |

---

## Practice Exercises

### Exercise 1: Basic Role Assignment

**Scenario:** You need to generate a PySpark function to validate transaction amounts.

#### 📝 Your Task

Write a prompt that assigns an appropriate role and requests the validation function.

**Hints:**
- What expertise is needed? (data engineering, validation)
- What domain knowledge? (banking, compliance)
- What specific requirements?

---

#### ✅ Solution

**Effective Prompt:**
```
> You are a senior data engineer specializing in financial data validation
> for banking transaction systems.
>
> Create a PySpark validation function for transaction amounts in
> pipelines/validators/amount_validator.py
>
> Requirements:
> - Input: DataFrame with transaction_schema (schemas/transaction.py)
> - Validation rules:
>   - amount must be Decimal(18, 2)
>   - amount must be positive (> 0)
>   - amount must not exceed $10,000 (daily limit per PCI-DSS policy)
> - Output: Tuple of (valid_df, invalid_df, validation_stats)
> - Logging: Log validation failures (mask account_id for PCI-DSS)
> - Testing: Include pytest test cases with sample data
>
> Follow the pattern in pipelines/validators/schema_validator.py
```

**Why This Works:**
- ✅ Role matches the task (data engineer + validation expertise)
- ✅ Domain context (banking, PCI-DSS)
- ✅ Specific requirements
- ✅ References existing patterns

---

### Exercise 2: Role for Code Review

**Scenario:** You want Claude to review a transaction processing pipeline for security issues.

#### 📝 Your Task

Assign a role that will provide a security-focused code review.

**Hints:**
- What perspective? (security, not just general development)
- What compliance standards?
- What should be reviewed?

**Code to Review:**
```python
def load_transactions(file_path: str) -> DataFrame:
    """Load transactions from parquet file."""
    df = spark.read.parquet(file_path)

    # Log for debugging
    print(f"Loaded {df.count()} transactions")
    df.show(10)

    return df
```

---

#### ✅ Solution

**Effective Prompt:**
```
> You are a PCI-DSS compliance officer responsible for reviewing banking
> data pipelines for security violations and data exposure risks.
>
> Review this transaction loading function from pipelines/loaders/transaction_loader.py:
>
> ```python
> def load_transactions(file_path: str) -> DataFrame:
>     """Load transactions from parquet file."""
>     df = spark.read.parquet(file_path)
>
>     # Log for debugging
>     print(f"Loaded {df.count()} transactions")
>     df.show(10)
>
>     return df
> ```
>
> Check for:
> - PCI-DSS violations (PAN exposure, CVV storage, logging sensitive data)
> - Missing data masking in logs
> - Audit trail gaps
> - Encryption requirements
>
> Provide:
> - List of security issues found
> - Risk level for each (High/Medium/Low)
> - Recommended fixes with code examples
> - References to PCI-DSS requirements
```

**Why This Works:**
- ✅ Role is security-focused (compliance officer)
- ✅ Specific compliance framework (PCI-DSS)
- ✅ Clear review criteria
- ✅ Structured output format

**Expected Findings:**
- **HIGH RISK:** `df.show(10)` exposes sensitive transaction data (account numbers, amounts)
- **HIGH RISK:** No data masking in logs
- **MEDIUM RISK:** No audit logging of data access
- **MEDIUM RISK:** No validation that CVV field is absent

---

### Exercise 3: Multiple Roles for Different Tasks

**Scenario:** You need to:
1. Design a new aggregation pipeline (architecture task)
2. Review it for compliance (security task)
3. Write tests for it (testing task)

#### 📝 Your Task

Write three separate prompts with appropriate roles for each task.

---

#### ✅ Solution

**Task 1: Design (Data Architect Role)**
```
> You are a senior data architect specializing in Delta Lake and data lakehouse
> design for banking applications.
>
> Design a daily account balance aggregation pipeline.
>
> Requirements:
> - Input: delta_lake.transactions table (100M+ rows)
> - Aggregation: Calculate end-of-day balance per account_id
> - Update frequency: Daily at 11 PM
> - Query pattern: Frequently queried by account_id and date
> - Retention: 7 years (SOX compliance)
>
> Provide:
> - Delta Lake table schema
> - Partitioning strategy
> - Merge/upsert approach for incremental updates
> - Query optimization recommendations
> - Sample PySpark code
```

**Task 2: Security Review (Compliance Officer Role)**
```
> You are a PCI-DSS and SOX compliance officer reviewing banking data pipelines.
>
> Review the daily account balance aggregation pipeline design:
> [paste design from Task 1]
>
> Check for:
> - PCI-DSS: Are account balances considered sensitive? Masking requirements?
> - SOX: Audit trail for balance calculations, change tracking, data lineage
> - GDPR: Data retention policy enforcement, right to deletion
> - Access controls: Who can read balance data?
>
> Provide:
> - Compliance gaps with severity (High/Medium/Low)
> - Recommended mitigations
> - Required audit logging
> - Data classification recommendations
```

**Task 3: Testing (Test Engineer Role)**
```
> You are a test automation engineer specializing in PySpark data pipeline testing.
>
> Write a comprehensive pytest test suite for the daily account balance
> aggregation pipeline.
>
> Test coverage requirements:
> 1. Unit tests:
>    - Balance calculation correctness
>    - Incremental update logic
>    - Edge cases (null amounts, negative balances, same-day corrections)
>
> 2. Integration tests:
>    - End-to-end pipeline execution
>    - Delta Lake merge/upsert correctness
>    - Partition pruning effectiveness
>
> 3. Data quality tests:
>    - No duplicate account_id + date combinations
>    - Balance matches sum of transaction amounts
>    - All accounts from source present in output
>
> File: tests/integration/test_account_balance_pipeline.py
> Use pyspark.testing.utils for DataFrame comparison
> Include test data fixtures in tests/fixtures/
```

**Why This Works:**
- ✅ Each role matches the task expertise
- ✅ Sequential workflow (design → review → test)
- ✅ Each role brings different perspective
- ✅ Comprehensive coverage of architecture, security, and quality

---

### Exercise 4: Combining Role with CLAUDE.md

**Scenario:** Your project has a data engineer role defined in CLAUDE.md, but you need a one-time performance audit.

#### 📝 Your Task

Write a prompt that temporarily assigns a performance specialist role while acknowledging the project's default role.

---

#### ✅ Solution

**Effective Prompt:**
```
> For this specific task, act as a Spark performance optimization specialist
> (temporarily setting aside your usual data engineer role).
>
> Conduct a performance audit of the transaction enrichment pipeline in
> pipelines/etl/enrich_transactions.py
>
> Current performance:
> - Execution time: 45 minutes
> - Data volume: 500K transactions/day
> - Cluster: 4 executors, 8GB memory each
> - Target: <20 minutes
>
> Analyze and report on:
> 1. Shuffle operations (identify unnecessary shuffles)
> 2. Join strategies (evaluate broadcast join opportunities)
> 3. Partition count optimization
> 4. Caching/persistence opportunities
> 5. Resource utilization (executor memory, cores)
>
> Provide:
> - Performance bottlenecks identified (with line numbers)
> - Estimated impact of each optimization (e.g., "20% reduction")
> - Prioritized optimization recommendations
> - Code examples for top 3 optimizations
> - Spark UI metrics to monitor
```

**Why This Works:**
- ✅ Explicitly overrides default role
- ✅ Explains temporary nature
- ✅ Assigns specialized role for specific task
- ✅ Provides complete context
- ✅ Specifies detailed output requirements

---

## Summary

In this subsection, you learned:

### Role Prompting Basics
- ✅ Roles shape perspective and output
- ✅ Different roles lead to different insights
- ✅ Roles set expectations for depth and focus

### Effective Role Assignment
- ✅ Be specific about expertise and experience
- ✅ Include relevant domain knowledge (banking, compliance)
- ✅ Specify the perspective (security, performance, quality)
- ✅ Combine role with task context and format

### Banking-Specific Roles
- ✅ Senior Data Engineer (pipeline design)
- ✅ PCI-DSS Compliance Officer (security reviews)
- ✅ Performance Specialist (optimization)
- ✅ Data Quality Engineer (validation)
- ✅ Test Engineer (test generation)
- ✅ Code Reviewer (quality checks)
- ✅ Data Architect (schema design)
- ✅ DevOps Engineer (CI/CD)

### Role Placement Strategies
- ✅ CLAUDE.md for consistent project-level role
- ✅ Inline prompts for task-specific roles
- ✅ Temporarily override default role when needed

---

## Key Takeaways

**When assigning roles:**

1. **Match role to task**
   - Security task → Security/Compliance role
   - Performance task → Optimization specialist role
   - Design task → Architect role

2. **Be specific**
   - "Senior data engineer" < "Senior data engineer with 10 years of PySpark and PCI-DSS experience"

3. **Include context**
   - Not just "who" but "why this role matters for this task"

4. **Consider multiple roles**
   - Complex tasks may benefit from multiple perspectives (design → review → test)

**Role Formula:**
```
You are a [LEVEL] [SPECIALTY] with [EXPERIENCE]
specializing in [DOMAIN] and [SPECIFIC SKILLS].
```

---

## Next Steps

👉 **[Continue to 1.4.5: Separating Data and Instructions](./04-05-separating-data-instructions.md)**

**Before proceeding:**
1. ✅ Review your CLAUDE.md - does it define a clear role?
2. ✅ Identify 3 tasks that would benefit from different roles
3. ✅ Practice assigning roles in your next prompts

---

**Related Sections:**
- [Phase 1.3.3: Memory & Context](./03-03-memory-context.md) - CLAUDE.md role definition
- [Phase 1.4.2: Basic Prompt Structure](./04-02-basic-prompt-structure.md) - Task + Context + Format
- [Phase 1.4.3: Being Clear and Direct](./04-03-being-clear-direct.md) - Clarity principles

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 02_Assigning_Roles_(Role_Prompting).ipynb
