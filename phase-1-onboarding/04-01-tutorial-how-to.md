# Phase 1.4.1: Tutorial How-To

**Learning Objectives:**
- Understand how to use this tutorial effectively
- Set up your practice environment
- Learn the interactive exercise format
- Understand banking data engineering context

**Time Commitment:** 10 minutes

**Prerequisites:** Phase 1.1-1.3 completed

---

## Table of Contents
1. [About This Tutorial](#about-this-tutorial)
2. [How to Use This Tutorial](#how-to-use-this-tutorial)
3. [Practice Environment Setup](#practice-environment-setup)
4. [Exercise Format](#exercise-format)
5. [Banking Data Engineering Context](#banking-data-engineering-context)

---

## About This Tutorial

### What This Tutorial Covers

This tutorial teaches **prompt engineering** - the skill of crafting effective instructions for Claude Code to generate high-quality data engineering code, analysis, and documentation.

**Based on:** Anthropic's Interactive Prompt Engineering Tutorial
**Adapted for:** Banking IT Data Engineering with PySpark
**Focus:** Production-grade data pipelines, compliance, security

### Why Prompt Engineering Matters

**Poor Prompt:**
```
> Fix the pipeline
```

**Claude's Confusion:** Which pipeline? What's broken? What should the fix accomplish?

**Good Prompt:**
```
> The customer transaction validation pipeline in pipelines/validators/transaction.py
> is failing when the amount field contains null values.
>
> Please:
> 1. Add null handling with appropriate default or rejection
> 2. Log validation failures to audit table
> 3. Add pytest test case for null amount scenario
> 4. Ensure PCI-DSS compliance (no sensitive data in logs)
```

**Result:** Claude generates precisely what you need in fewer iterations.

---

## How to Use This Tutorial

### Learning Path

**Each subsection follows this structure:**

```
1. Concept Introduction
   ↓
2. Banking Data Engineering Example
   ↓
3. Bad Prompt (What NOT to do)
   ↓
4. Good Prompt (Effective approach)
   ↓
5. Claude's Response (Expected output)
   ↓
6. Practice Exercise (Try it yourself)
   ↓
7. Solution & Explanation
```

### Interactive Learning

**Do NOT just read** - actively practice each technique:

1. **Read the concept** (5 min)
2. **Study the examples** (5 min)
3. **Try the exercise** (10-15 min)
4. **Compare with solution** (5 min)
5. **Apply to your own code** (bonus)

### Time Commitment

| Subsection | Topic | Time |
|-----------|-------|------|
| 04-01 | Tutorial How-To | 10 min |
| 04-02 | Basic Prompt Structure | 30 min |
| 04-03 | Being Clear and Direct | 30 min |
| 04-04 | Assigning Roles | 30 min |
| 04-05 | Separating Data/Instructions | 30 min |
| 04-06 | Formatting Output | 30 min |
| 04-07 | Step-by-Step Reasoning | 45 min |
| 04-08 | Using Examples | 45 min |
| 04-09 | Avoiding Hallucinations | 30 min |
| 04-10 | Complex Prompts | 45 min |
| 04-11 | Chaining Prompts | 30 min |
| 04-12 | Tool Use | 30 min |
| 04-13 | Search & Retrieval | 30 min |

**Total:** ~4-5 hours (spread over multiple sessions)

---

## Practice Environment Setup

### Prerequisites

Before starting:

```bash
# 1. Verify Claude Code installation
claude --version
# Expected: claude-code version 0.x.x or higher

# 2. Verify Python and PySpark
python --version
# Expected: Python 3.10 or higher

# 3. Verify you can start Claude
claude
# Should open interactive session
# Type Ctrl+D to exit
```

### Create Practice Project

```bash
# Navigate to your workspace
cd ~/workspace

# Create practice project
mkdir prompt-engineering-practice
cd prompt-engineering-practice

# Create directory structure
mkdir -p pipelines/{etl,validators,transformers}
mkdir -p schemas
mkdir -p tests/{unit,integration}
mkdir -p config
mkdir -p .claude/{commands,hooks}

# Create .gitignore
cat > .gitignore << 'EOF'
.claude/settings.local.json
__pycache__/
*.pyc
.venv/
.pytest_cache/
.coverage
EOF

# Initialize git (optional)
git init
```

### Create Claude Code Configuration

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  },
  "defaultModel": "sonnet",
  "env": {
    "ENVIRONMENT": "practice",
    "PYSPARK_PYTHON": "python3"
  }
}
```

### Create Project Memory

`.claude/CLAUDE.md`:
```markdown
# Prompt Engineering Practice Project

## Your Role
You are a senior banking data engineer practicing prompt engineering techniques.

## Technology Stack
- Python 3.10+
- PySpark 3.5+
- pytest for testing
- Focus on learning effective prompts

## Practice Guidelines
- All examples use banking transaction data
- Focus on PCI-DSS compliance patterns
- Generate production-grade code
- Include comprehensive error handling
- Add type hints and docstrings

## Sample Data
Use this sample transaction data for all exercises:
```python
sample_transactions = [
    {"txn_id": "TXN001", "account_id": "ACC123", "amount": 100.50, "currency": "USD"},
    {"txn_id": "TXN002", "account_id": "ACC456", "amount": 250.00, "currency": "USD"},
    {"txn_id": "TXN003", "account_id": "ACC789", "amount": -50.00, "currency": "USD"}  # Invalid
]
```
```

### Verify Setup

```bash
# Start Claude Code
claude

# Test basic prompt
> Hello, I'm ready to practice prompt engineering for data engineering.
> What sample transaction data should I use for exercises?

# Expected: Claude should reference the CLAUDE.md context
# and mention the sample_transactions data

# Exit
Ctrl+D
```

---

## Exercise Format

### Exercise Structure

Each exercise in this tutorial follows this format:

#### 📝 Exercise: [Topic Name]

**Scenario:** [Banking data engineering context]

**Your Task:** [What you need to accomplish]

**Hints:**
- [Hint 1]
- [Hint 2]

**Try it:** [Specific prompt to attempt]

---

#### ✅ Solution

**Effective Prompt:**
```
[Example of a good prompt]
```

**Why This Works:**
- [Explanation point 1]
- [Explanation point 2]

**Claude's Response:**
```python
[Example code output]
```

### Practice Tips

**1. Try Before Looking at Solution**

Don't peek at the solution immediately. Attempt the exercise first, even if your prompt isn't perfect. Learning happens through trial and error.

**2. Compare Your Prompt**

After trying, compare your prompt with the solution:
- Did you provide enough context?
- Was your request clear and specific?
- Did you specify the output format?

**3. Iterate and Improve**

If your first prompt didn't work well:
- Add more context
- Be more specific
- Break into smaller steps
- Reference examples

**4. Apply to Real Code**

After completing an exercise, try the same technique on your actual banking data pipeline projects.

---

## Banking Data Engineering Context

### Sample Transaction Dataset

All exercises use this consistent dataset:

```python
# pipelines/sample_data.py
from decimal import Decimal
from datetime import datetime
from typing import List, Dict, Any

sample_transactions: List[Dict[str, Any]] = [
    {
        "txn_id": "TXN001",
        "account_id": "ACC123",
        "amount": Decimal("100.50"),
        "currency": "USD",
        "timestamp": datetime(2024, 1, 15, 10, 30, 0),
        "merchant_id": "MERCH001",
        "status": "completed"
    },
    {
        "txn_id": "TXN002",
        "account_id": "ACC456",
        "amount": Decimal("250.00"),
        "currency": "USD",
        "timestamp": datetime(2024, 1, 15, 11, 45, 0),
        "merchant_id": "MERCH002",
        "status": "completed"
    },
    {
        "txn_id": "TXN003",
        "account_id": "ACC789",
        "amount": Decimal("-50.00"),  # Invalid negative amount
        "currency": "USD",
        "timestamp": datetime(2024, 1, 15, 12, 15, 0),
        "merchant_id": "MERCH003",
        "status": "pending"
    },
    {
        "txn_id": "TXN004",
        "account_id": "ACC123",
        "amount": None,  # Invalid null amount
        "currency": "EUR",
        "timestamp": datetime(2024, 1, 15, 13, 20, 0),
        "merchant_id": "MERCH001",
        "status": "failed"
    }
]
```

### PySpark Schema

```python
from pyspark.sql.types import StructType, StructField, StringType, DecimalType, TimestampType

transaction_schema = StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("currency", StringType(), nullable=False),
    StructField("timestamp", TimestampType(), nullable=False),
    StructField("merchant_id", StringType(), nullable=True),
    StructField("status", StringType(), nullable=False)
])
```

### Common Banking Scenarios

Throughout the tutorial, you'll practice prompts for:

1. **Data Validation**
   - Schema enforcement
   - Business rule validation (amount > 0, valid currency)
   - Duplicate detection

2. **Data Transformation**
   - Currency conversion
   - Aggregations (daily totals, account balances)
   - Window functions (running balances)

3. **Compliance**
   - PCI-DSS: Masking sensitive data
   - SOX: Audit trail generation
   - GDPR: Data retention policies

4. **Data Quality**
   - Null handling
   - Referential integrity checks
   - Completeness validation

5. **Performance**
   - Partition optimization
   - Broadcast joins
   - Caching strategies

---

## What Makes a Good Prompt?

### The Three Pillars

Every good prompt needs:

1. **TASK**: What do you want Claude to do?
2. **CONTEXT**: What information does Claude need?
3. **FORMAT**: How should the output be structured?

### Example Comparison

**❌ Bad Prompt:**
```
> Create a validation function
```

**✅ Good Prompt:**
```
> Create a PySpark validation function for banking transactions

Input: DataFrame with transaction_schema (defined in schemas/transaction.py)
Validation rules:
- amount must be positive (> 0)
- amount must not exceed $10,000 (daily limit)
- currency must be USD, EUR, or GBP
- txn_id must be unique (no duplicates)

Output: Tuple of (valid_df, invalid_df)
Logging: Log all validation failures with txn_id (but mask account_id)
Testing: Include pytest test cases for each validation rule

Follow the pattern in pipelines/validators/account_validator.py
```

**Why Good Prompt Works:**
- ✅ Clear task: validation function
- ✅ Specific context: PySpark, schema, rules
- ✅ Output format: tuple of DataFrames
- ✅ Additional requirements: logging, testing
- ✅ Reference pattern: existing code

---

## Quick Reference Card

### Prompt Structure Template

```
> [ACTION VERB] [WHAT]

Context:
- [Relevant background]
- [Technology/framework]
- [Existing patterns to follow]

Requirements:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

Constraints:
- [Limitation 1]
- [Limitation 2]

Output:
- [Expected format]
- [Include tests/docs]
```

### Common Action Verbs for Data Engineering

| Action | Use For |
|--------|---------|
| Generate | Creating new pipelines, schemas, tests |
| Refactor | Improving existing code structure |
| Debug | Finding and fixing issues |
| Optimize | Performance improvements |
| Validate | Adding data quality checks |
| Document | Creating documentation |
| Analyze | Understanding code/data |
| Transform | Data manipulation logic |

---

## Summary

In this subsection, you learned:

### Tutorial Format
- ✅ Interactive exercises with banking scenarios
- ✅ Practice → Compare → Learn approach
- ✅ 13 subsections covering core to advanced techniques

### Setup
- ✅ Created practice environment
- ✅ Configured Claude Code settings
- ✅ Set up project memory (CLAUDE.md)

### Good Prompts Need
- ✅ Clear task description
- ✅ Relevant context
- ✅ Specified output format
- ✅ Banking domain requirements

### Practice Dataset
- ✅ Sample transactions for all exercises
- ✅ PySpark schema definitions
- ✅ Common banking scenarios

---

## Next Steps

👉 **[Continue to 1.4.2: Basic Prompt Structure](./04-02-basic-prompt-structure.md)**

**Before proceeding:**
1. ✅ Complete practice environment setup
2. ✅ Verify Claude Code is working
3. ✅ Review the sample transaction data
4. ✅ Understand the exercise format

---

**Related Sections:**
- [Phase 1.3.3: Memory & Context](./03-03-memory-context.md) - CLAUDE.md setup
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Settings
- [Phase 1.4 Index](./04-00-prompt-engineering-index.md) - Tutorial overview

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 00_Tutorial_How-To.ipynb
