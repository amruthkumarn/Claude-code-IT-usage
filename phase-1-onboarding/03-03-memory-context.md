# Phase 1.3.3: Memory & Context Management

**Learning Objectives:**
- Understand types of memory in Claude Code
- Learn memory hierarchy and precedence
- Master CLAUDE.md for project standards
- Implement effective context management

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.3.1 and 1.3.2 completed

---

## ⚡ Quick Start (5 minutes)

**Goal:** Create your first CLAUDE.md and watch Claude use it automatically.

### Try This Right Now

```bash
# 1. Create a simple CLAUDE.md
cd ~/practice-project
cat > CLAUDE.md << 'EOF'
# My Practice Project

## Coding Standards
- Use descriptive variable names
- Add comments for complex logic
- Follow PEP 8 style guide
EOF

# 2. Start Claude Code
claude

# 3. Test if Claude reads it
> Generate a Python function to calculate the sum of two numbers
```

**What you'll see:**
- Claude generates code with **descriptive variable names** ✅
- Claude adds **comments** ✅
- Claude follows **PEP 8** ✅
- All without you mentioning these requirements!

**What just happened?**
- Claude automatically read CLAUDE.md at startup
- Your standards are now embedded in Claude's context
- Every code generation follows your rules

**Next:** Let's understand how memory works in detail...

---

## Table of Contents
1. [What is Memory in Claude Code?](#what-is-memory-in-claude-code)
2. [Memory Hierarchy](#memory-hierarchy)
3. [CLAUDE.md: Project Memory](#claudemd-project-memory)
4. [Banking IT CLAUDE.md Template](#banking-it-claudemd-template)
5. [How Claude Uses CLAUDE.md](#how-claude-uses-claudemd)
6. [Context Management Tips](#context-management-tips)

---

## What is Memory in Claude Code?

**Memory** = Information Claude remembers about your project

### Types of Memory

**1. Session Memory** (Temporary)
- Lasts for one session (conversation)
- Current tool uses and responses
- Recent file reads and edits
- **Duration:** Until you exit (`Ctrl+D` or `/exit`)

**Example:**
```
> Read the payment pipeline

[Later in same session]
> Add error handling to that pipeline
# Claude remembers "that pipeline" = payment_pipeline.py
```

**2. Project Memory** (Persistent)
- Stored in `CLAUDE.md` or `.claude/CLAUDE.md`
- Team-shared standards and conventions
- Coding patterns and best practices
- **Duration:** Permanent (committed to git)

**Example:**
```markdown
# CLAUDE.md
## Coding Standards
- Use Decimal(18,2) for all monetary amounts
- Always mask account_number in non-production logs
```

**3. User Memory** (Personal)
- Stored in `~/.claude/CLAUDE.md`
- Personal preferences across all projects
- Common patterns you prefer
- **Duration:** Permanent (personal only)

---

## Memory Hierarchy

```
┌─────────────────────────────────────────────┐
│ 1. SESSION MEMORY (Highest Priority)       │
│    - Current conversation                    │
│    - Recent tool uses                        │
│    - Temporary context                       │
│    Duration: Until you exit                  │
└─────────────────────────────────────────────┘
           ↓ (if not in session)
┌─────────────────────────────────────────────┐
│ 2. PROJECT MEMORY (CLAUDE.md)              │
│    - Project standards                       │
│    - Coding conventions                      │
│    - Team preferences                        │
│    Location: ./CLAUDE.md or .claude/CLAUDE.md│
└─────────────────────────────────────────────┘
           ↓ (if not in project)
┌─────────────────────────────────────────────┐
│ 3. USER MEMORY (~/.claude/CLAUDE.md)       │
│    - Personal preferences                    │
│    - Common patterns                         │
│    - Global standards                        │
│    Location: ~/.claude/CLAUDE.md            │
└─────────────────────────────────────────────┘
```

**Precedence Rule:** Session > Project > User

---

## CLAUDE.md: Project Memory

### Purpose

Document your project's standards so Claude automatically follows them.

### Location

Choose one:
- `./CLAUDE.md` (project root) - **Recommended** (more visible)
- `./.claude/CLAUDE.md` (hidden directory) - Alternative

**Both work the same way. Choose based on team preference.**

### What to Include

1. **Project Overview** - What this project does
2. **Coding Standards** - PySpark conventions, naming
3. **Security Requirements** - PII handling, compliance
4. **Testing Strategy** - pytest patterns, coverage goals
5. **Common Patterns** - Reusable code patterns
6. **Team Conventions** - PR process, documentation style

---

## Banking IT CLAUDE.md Template

### Complete Example

**File:** `CLAUDE.md` (in project root)

```markdown
# Payment Processing Pipeline

## Project Overview
PySpark-based payment transaction processing pipeline for banking operations.

**Tech Stack:**
- Python 3.9+
- PySpark 3.4+
- Delta Lake 2.4+
- pytest, Ruff, Black, mypy

**Data Flow:**
S3 Raw → Bronze (validated) → Silver (cleansed) → Gold (aggregated)

## Coding Standards

### Python/PySpark
- Use type hints on all function signatures
- Follow PEP 8 (enforced by Ruff)
- Format with Black (line length: 100)
- Type check with mypy
- Minimum Python 3.9

### PySpark Patterns
```python
# Preferred: Explicit schema
from pyspark.sql.types import StructType, StructField, StringType, DecimalType

schema = StructType([
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False)
])

df = spark.read.schema(schema).parquet("s3://...")

# NOT: Inferred schema (too risky for production)
```

### Naming Conventions
- Functions: `snake_case` (e.g., `transform_customer_data`)
- Classes: `PascalCase` (e.g., `PaymentProcessor`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`)
- Files: `snake_case.py` (e.g., `payment_validator.py`)

## Security Requirements

### PII Handling
**ALWAYS mask PII in non-production environments:**
```python
def mask_account_number(account_num: str) -> str:
    """Mask account number for PCI-DSS compliance."""
    return f"****{account_num[-4:]}"
```

### Secrets Management
- **NEVER** hardcode credentials
- Use environment variables or AWS Secrets Manager
- Validate secrets at startup:
```python
import os

DB_PASSWORD = os.getenv("DB_PASSWORD")
if not DB_PASSWORD:
    raise ValueError("DB_PASSWORD environment variable required")
```

### Compliance Checks
- **PCI-DSS**: No card data in logs or temp storage
- **SOX**: Audit trail for all financial transactions
- **GDPR**: Personal data properly masked/anonymized

## Testing Standards

### pytest Conventions
```python
# Test file naming: test_*.py or *_test.py
# tests/test_payment_validator.py

import pytest
from pyspark.sql import SparkSession

@pytest.fixture(scope="session")
def spark():
    return SparkSession.builder.master("local[2]").getOrCreate()

def test_validate_payment_amount(spark):
    # Arrange
    data = [("TX001", -100.0)]  # Invalid: negative amount
    df = spark.createDataFrame(data, ["tx_id", "amount"])

    # Act
    result = validate_payment_amount(df)

    # Assert
    assert result.count() == 0  # Invalid records filtered out
```

### Code Coverage
- **Minimum:** 80% coverage
- **Target:** 90%+ coverage
- Run: `pytest --cov=pipelines --cov-report=html`

## Common Patterns

### Data Quality Validation
```python
def validate_dataframe(df: DataFrame, required_cols: List[str]) -> DataFrame:
    """
    Validate DataFrame has required columns and no nulls.

    Raises:
        ValueError: If validation fails
    """
    # Check schema
    missing_cols = set(required_cols) - set(df.columns)
    if missing_cols:
        raise ValueError(f"Missing columns: {missing_cols}")

    # Check nulls
    null_counts = df.select([
        F.count(F.when(F.col(c).isNull(), c)).alias(c)
        for c in required_cols
    ]).first().asDict()

    if any(count > 0 for count in null_counts.values()):
        raise ValueError(f"Null values found: {null_counts}")

    return df
```

### Error Handling
```python
import logging

logger = logging.getLogger(__name__)

def process_payments(df: DataFrame) -> DataFrame:
    """Process payment transactions with error handling."""
    try:
        # Transformation logic
        result = df.filter(F.col("status") == "PENDING") \
                   .withColumn("processed_at", F.current_timestamp())

        logger.info(f"Processed {result.count()} payments")
        return result

    except Exception as e:
        logger.error(f"Payment processing failed: {str(e)}", exc_info=True)
        raise
```

## Team Conventions

### Git Commit Messages
```
feat(pipelines): add daily payment aggregation

- Implement aggregate_payments_by_merchant()
- Add data quality checks
- Include unit tests with 95% coverage

Compliance: PCI-DSS data masking implemented
```

### Pull Request Template
1. **Summary**: What changed and why?
2. **Testing**: How was this tested?
3. **Compliance**: Any PCI/SOX/GDPR impacts?
4. **Performance**: Any performance implications?
5. **Rollback**: How to rollback if needed?

### Documentation
- Every public function has docstring
- Complex logic has inline comments
- README updated for significant changes

## Don't Do This

### Anti-Patterns
```python
# ❌ BAD: Hardcoded credentials
DB_HOST = "prod-db.bank.internal"
DB_PASSWORD = "password123"

# ❌ BAD: No type hints
def process(data):
    return data.filter(...)

# ❌ BAD: Generic exception handling
try:
    process_payments()
except:
    pass

# ❌ BAD: No schema validation
df = spark.read.parquet("s3://...")  # Inferred schema = risk

# ❌ BAD: Print statements (use logging)
print(f"Processing {count} records")
```

### Data Quality
```python
# ❌ BAD: No validation
result = df.groupBy("merchant_id").agg(F.sum("amount"))

# ✅ GOOD: Validate before aggregation
result = df.filter(F.col("amount").isNotNull()) \
           .filter(F.col("amount") > 0) \
           .groupBy("merchant_id") \
           .agg(F.sum("amount").alias("total_amount"))

# Verify results
assert result.filter(F.col("total_amount") < 0).count() == 0
```

## Performance Guidelines

### PySpark Optimization Checklist
- [ ] Use explicit schemas (StructType)
- [ ] Partition data appropriately
- [ ] Broadcast small dimension tables
- [ ] Cache/persist reused DataFrames
- [ ] Avoid shuffles when possible
- [ ] Filter early, aggregate late
- [ ] Use column pruning

### Example Optimization
```python
# Before: Multiple shuffles
df = large_df.join(small_df, "key")  # Shuffle!
result = df.groupBy("merchant").agg(...)  # Another shuffle!

# After: Broadcast join + partition-aware aggregation
from pyspark.sql.functions import broadcast

df = large_df.join(broadcast(small_df), "key")  # No shuffle
result = df.repartition("merchant") \
           .groupBy("merchant") \
           .agg(...)  # Optimized shuffle
```

---
```

**Save this as:** `CLAUDE.md` in your project root

---

## 🔨 Exercise 1: Create Your Banking IT CLAUDE.md (15 minutes)

**Goal:** Build a complete CLAUDE.md for a banking data engineering project.

### Step 1: Create the project structure

```bash
mkdir -p ~/banking-pipeline-project/{pipelines,tests,schemas,config}
cd ~/banking-pipeline-project
```

### Step 2: Create your CLAUDE.md

```bash
cat > CLAUDE.md << 'EOF'
# Banking Transaction Pipeline

## Project Overview
PySpark-based ETL pipeline for processing daily banking transactions.
Handles 500K+ transactions/day with PCI-DSS and SOX compliance.

**Tech Stack:**
- Python 3.10+
- PySpark 3.5+
- Delta Lake 2.4+
- pytest, pytest-spark

## Coding Standards

### Type Hints Required
```python
# Always use type hints
def process_transaction(amount: Decimal, account_id: str) -> bool:
    pass
```

### Naming Conventions
- Functions: `snake_case` (e.g., `validate_account`)
- Classes: `PascalCase` (e.g., `TransactionProcessor`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_AMOUNT`)

### Money Handling
```python
# ALWAYS use Decimal for money
from decimal import Decimal
amount = Decimal("100.50")  # ✅ Correct
amount = 100.50  # ❌ NEVER use float for money
```

## Security Requirements

### PII Masking
```python
# Mask account numbers in logs
def mask_account(account: str) -> str:
    return f"****{account[-4:]}"
```

### No Hardcoded Secrets
```python
# ✅ Correct
DB_PASSWORD = os.getenv("DB_PASSWORD")

# ❌ NEVER do this
DB_PASSWORD = "password123"
```

## Testing Standards
- Minimum 80% code coverage
- Use pytest with SparkSession fixtures
- Test happy path, edge cases, and error conditions

## Banking-Specific Rules
1. All transactions must have audit trail
2. Amounts validated before processing
3. Failed transactions logged separately
4. Data retention: 7 years (SOX requirement)
EOF
```

### Step 3: Test it with Claude

```bash
claude

> Generate a PySpark function to validate transaction amounts
```

### Step 4: Verify Claude follows your standards

**Check the generated code for:**
- ✅ Type hints present?
- ✅ Uses `Decimal` for amounts?
- ✅ Function name in snake_case?
- ✅ Has proper docstring?
- ✅ Includes validation logic?

### Step 5: Test banking-specific requirements

```
> Generate a function to log a transaction to an audit table

Requirements:
- Include timestamp
- Include user information
- Mask sensitive PII
```

**Expected:** Claude should automatically:
- Mask account numbers (from PII Masking section)
- Include audit trail logic (from Banking-Specific Rules)
- Use proper naming conventions

### ✅ Checkpoint
- [ ] You created a comprehensive CLAUDE.md
- [ ] Claude automatically followed your standards
- [ ] Generated code used Decimal for money
- [ ] Generated code included PII masking
- [ ] All without explicitly mentioning these in your prompt

### 💻 Terminal Session Example

```bash
$ claude

> Generate a function to validate transaction amounts

🤖 I'll create a validation function following your project standards.

🔧 Tool Use: Write
File: pipelines/validators/transaction_validator.py

```python
from decimal import Decimal
from typing import Tuple
from pyspark.sql import DataFrame
import pyspark.sql.functions as F

def validate_transaction_amount(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """
    Validate transaction amounts according to banking rules.

    Args:
        df: Input DataFrame with 'amount' column (DecimalType)

    Returns:
        Tuple of (valid_df, invalid_df)

    Banking Rules:
    - Amount must be positive (> 0)
    - Amount must not exceed daily limit
    - Proper decimal precision
    """
    # Validate positive amounts
    valid_df = df.filter(F.col("amount") > Decimal("0"))
    invalid_df = df.filter(F.col("amount") <= Decimal("0"))

    return valid_df, invalid_df
```

Approve this write? (yes/no) yes

✅ Created: pipelines/validators/transaction_validator.py
```

**Notice:**
- Uses `Decimal` (from your standards) ✅
- Type hints present ✅
- snake_case naming ✅
- Comprehensive docstring ✅
- Banking-specific validation ✅

### 🎯 Challenge: Add More Standards

Try adding these to your CLAUDE.md:
1. Logging format requirements
2. Error handling patterns
3. Performance optimization rules

Then ask Claude to generate code and see if it follows them!

<details>
<summary>💡 Example Addition</summary>

```markdown
## Logging Standards
```python
import logging
logger = logging.getLogger(__name__)

# Always use structured logging
logger.info("Processing transaction", extra={
    "transaction_id": txn_id,
    "amount": str(amount),  # Convert Decimal to string
    "account_id": mask_account(account_id)  # Mask PII
})
```
```

</details>

**Key Insight:** CLAUDE.md is your "always-on" project documentation. Claude reads it automatically and follows every rule, every time.

---

## How Claude Uses CLAUDE.md

### Automatic Reading

When you start Claude Code, it automatically:
1. ✅ Reads `CLAUDE.md` (if it exists)
2. ✅ Incorporates standards into context
3. ✅ Follows conventions in generated code

**You don't need to tell Claude to read it—it happens automatically!**

### Example: Code Generation

**Without CLAUDE.md:**
```
> Generate a function to validate payment amounts

# Claude generates generic code with no specific standards
def validate_payment(amount):
    if amount > 0:
        return True
    return False
```

**With CLAUDE.md:**
```
> Generate a function to validate payment amounts

# Claude reads CLAUDE.md and generates compliant code:
def validate_payment_amount(df: DataFrame) -> DataFrame:
    """
    Validate payment amounts for PCI-DSS compliance.

    Rules:
    - Amount must be > 0
    - Amount must be <= 1,000,000 (fraud detection)
    - Must have proper decimal precision

    Args:
        df: Input DataFrame with 'amount' column

    Returns:
        Validated DataFrame (invalid records filtered)

    Raises:
        ValueError: If 'amount' column missing
    """
    # Follows standards from CLAUDE.md:
    # - Type hints ✓
    # - Docstring ✓
    # - Validation logic ✓
    # - PCI-DSS compliant ✓

    if "amount" not in df.columns:
        raise ValueError("Missing 'amount' column")

    return df.filter(F.col("amount") > 0) \
             .filter(F.col("amount") <= 1_000_000)
```

**Key Differences:**
- ✅ Type hints (`DataFrame` annotations)
- ✅ Comprehensive docstring
- ✅ PySpark DataFrame API
- ✅ Proper validation and error handling
- ✅ Follows naming conventions

---

## Context Management Tips

### 1. Keep Sessions Focused

**Don't:**
```bash
> Fix payment validation
> Now update customer schema
> Actually, analyze transaction trends
> Wait, debug the aggregation function
```

**Do:**
```bash
> Fix payment validation
> Add unit tests for validation
> Run tests and verify
[Complete this workflow, then start new session for next task]
```

**Why:** Focused sessions maintain better context

### 2. Start Fresh When Needed

```bash
# If Claude "forgets" context or seems confused
Ctrl+D  # Exit
claude  # New session, fresh context
```

**When to restart:**
- After 20-30 interactions
- When switching to unrelated task
- If Claude seems to lose context

### 3. Use CLAUDE.md for Persistent Knowledge

**Add to CLAUDE.md when you want Claude to remember:**
- ✅ Project-specific patterns
- ✅ Team conventions
- ✅ Common pitfalls to avoid
- ✅ Performance optimization rules

**Don't rely on session memory for:**
- ❌ Long-term project standards
- ❌ Information needed across sessions
- ❌ Team-shared conventions

### 4. Leverage Session Memory

```
> Read the payment pipeline code

[Later in same session]
> Add error handling to that pipeline
# Claude remembers which pipeline

> Now generate tests for it
# Claude still remembers the pipeline context
```

**Best Practice:** Chain related tasks in one session

---

## 🔨 Exercise 2: Test Session Memory (10 minutes)

**Goal:** Understand how Claude remembers context within a session.

### Step 1: Start a fresh Claude Code session

```bash
cd ~/banking-pipeline-project
claude
```

### Step 2: Establish context

```
> Read the file pipelines/validators/transaction_validator.py
```

**Claude reads the file and now has it in session memory.**

### Step 3: Test implicit references

```
> Add a function to that file to validate account IDs
```

**Expected:** Claude knows "that file" = transaction_validator.py (from Step 2)

### Step 4: Test chained context

```
> Now generate tests for both functions in that file
```

**Expected:** Claude remembers:
- "that file" = transaction_validator.py
- "both functions" = validate_transaction_amount + validate_account_id

### Step 5: Test context limits

```
> Tell me about the payment processing module
```

**Expected:** Claude will search for it (not in session memory yet)

```
> Now optimize that module
```

**Expected:** Claude knows "that module" = the payment processing file it just found

### ✅ Checkpoint
- [ ] You established context with Read
- [ ] You used implicit references ("that file")
- [ ] You chained multiple related tasks
- [ ] Claude remembered context throughout the session

### 💡 Session Memory Best Practices

**✅ Do: Chain related tasks**
```
> Read pipeline.py
> Add error handling to that pipeline
> Generate tests for it
> Run the tests
```
All in one session - Claude maintains context.

**❌ Don't: Mix unrelated tasks**
```
> Read pipeline.py
> Create a new schema for customers  # Unrelated!
> Update that pipeline  # Which one? Context lost!
```

### 🎯 Challenge: Test Context Boundaries

Exit Claude (`Ctrl+D`) and restart. Then try:
```
> Update that file
```

**Question:** What happens?

<details>
<summary>💡 Answer</summary>

Claude will ask **"Which file?"** because session memory was cleared when you exited.

**Lesson:** Session memory is temporary. Use CLAUDE.md for persistent knowledge.

</details>

**Key Insight:** Session memory is powerful for focused workflows but doesn't persist across restarts. Use CLAUDE.md for long-term project knowledge.

---

### 5. Update CLAUDE.md Regularly

```bash
# After establishing new patterns
> I've added a new validation pattern to CLAUDE.md

# Claude will re-read it in next session
Ctrl+D
claude
# New session picks up updated CLAUDE.md
```

---

## Practical Example

### Scenario: New Team Member Onboarding

**Without CLAUDE.md:**
- New developer asks Claude for help
- Gets generic Python/PySpark code
- Has to manually apply team standards
- Inconsistent code across team

**With CLAUDE.md:**
- New developer starts Claude Code
- Claude automatically reads CLAUDE.md
- Generated code follows team standards
- Consistent code from day one

### Code Example

**Developer asks:**
```
> Generate a function to read payment transactions from S3
```

**Claude generates (using CLAUDE.md standards):**
```python
from pyspark.sql import SparkSession, DataFrame
from pyspark.sql.types import StructType, StructField, StringType, DecimalType, TimestampType
import logging

logger = logging.getLogger(__name__)

def read_payment_transactions(
    spark: SparkSession,
    s3_path: str
) -> DataFrame:
    """
    Read payment transactions from S3 with explicit schema.

    Args:
        spark: Active SparkSession
        s3_path: S3 path to payment data (e.g., s3://banking-data/payments/)

    Returns:
        DataFrame with validated payment transaction schema

    Raises:
        ValueError: If data doesn't match expected schema
    """
    # Explicit schema (from CLAUDE.md standard)
    schema = StructType([
        StructField("transaction_id", StringType(), nullable=False),
        StructField("account_id", StringType(), nullable=False),
        StructField("amount", DecimalType(18, 2), nullable=False),
        StructField("timestamp", TimestampType(), nullable=False),
        StructField("status", StringType(), nullable=False)
    ])

    try:
        df = spark.read.schema(schema).parquet(s3_path)
        logger.info(f"Successfully read {df.count()} payment transactions from {s3_path}")
        return df
    except Exception as e:
        logger.error(f"Failed to read payment transactions: {str(e)}", exc_info=True)
        raise
```

**Automatically includes:**
- ✅ Type hints
- ✅ Docstring
- ✅ Explicit schema (StructType)
- ✅ DecimalType for amounts
- ✅ Logging instead of print
- ✅ Error handling
- ✅ Team conventions

---

## Summary

In this subsection, you learned:

### Memory Types
- ✅ Session memory (temporary, current conversation)
- ✅ Project memory (CLAUDE.md, team-shared)
- ✅ User memory (~/.claude/CLAUDE.md, personal)

### Memory Hierarchy
- ✅ Session > Project > User
- ✅ Most specific wins

### CLAUDE.md
- ✅ Project standards document
- ✅ Automatically read by Claude
- ✅ Enforces team conventions
- ✅ Banking IT template provided

### Context Management
- ✅ Keep sessions focused
- ✅ Start fresh when needed
- ✅ Use CLAUDE.md for persistence
- ✅ Leverage session memory for related tasks

---

## Next Steps

👉 **[Continue to 1.3.4: CLI Reference & Commands](./03-04-cli-reference.md)**

**Quick Practice:**
1. Create `CLAUDE.md` in your project using the banking IT template
2. Start Claude Code and ask it to generate a function
3. Verify it follows your CLAUDE.md standards

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Settings files
- [Phase 1.3.11: Standards & Best Practices](./03-11-standards-best-practices.md) - More standards
- [Phase 1.1: Introduction](./01-introduction-getting-started.md) - Workflows

---

**Last Updated:** 2025-10-23
**Version:** 1.0
