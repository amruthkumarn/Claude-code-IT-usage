# Phase 1.4.2: Basic Prompt Structure

**Learning Objectives:**
- Master the Task + Context + Format formula
- Apply structured prompts to data engineering
- Generate PySpark pipelines with clear instructions
- Understand the impact of prompt structure on output quality

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1 completed

---

## Table of Contents
1. [The Golden Formula](#the-golden-formula)
2. [Task: What to Do](#task-what-to-do)
3. [Context: Background Information](#context-background-information)
4. [Format: Output Structure](#format-output-structure)
5. [Data Engineering Examples](#data-engineering-examples)
6. [Practice Exercises](#practice-exercises)

---

## The Golden Formula

### Task + Context + Format = Effective Prompt

```
PROMPT = TASK + CONTEXT + FORMAT

Where:
- TASK: What you want Claude to do
- CONTEXT: Information Claude needs to know
- FORMAT: How the output should be structured
```

### Why This Works

**Without Structure:**
- Claude makes assumptions
- Output varies wildly
- Multiple iterations needed
- May miss requirements

**With Structure:**
- Clear expectations
- Consistent output
- Fewer iterations
- Predictable results

---

## Task: What to Do

### Start with a Clear Action Verb

**Vague:**
```
> Do something with the transaction data
```

**Clear:**
```
> Validate transaction data for completeness and business rules
```

### Common Data Engineering Tasks

| Task Verb | Data Engineering Use |
|-----------|---------------------|
| **Generate** | Create new pipeline, schema, or transformation |
| **Validate** | Add data quality checks and validation logic |
| **Transform** | Modify data structure or values |
| **Aggregate** | Summarize data (GROUP BY, window functions) |
| **Join** | Combine datasets with merge logic |
| **Debug** | Find and fix issues in pipelines |
| **Optimize** | Improve performance (partitioning, caching) |
| **Test** | Create pytest test cases |
| **Document** | Generate documentation or comments |

### Be Specific About Scope

**Too Broad:**
```
> Fix the payment pipeline
```

**Specific:**
```
> Fix the null pointer exception in pipelines/payment_processing.py
> at line 142 when processing transactions with missing merchant_id
```

---

## Context: Background Information

### What Context to Provide

1. **Technology Stack**
   - Language: Python, PySpark
   - Frameworks: Delta Lake, Great Expectations
   - Version: PySpark 3.5+

2. **Data Schema**
   - Input schema (StructType)
   - Output schema
   - Sample data

3. **Business Rules**
   - Validation logic
   - Transformation requirements
   - Compliance needs

4. **Existing Patterns**
   - Reference implementations
   - Code style
   - Project conventions

### Example: Providing Schema Context

**Without Context:**
```
> Create a validation function
```

**With Schema Context:**
```
> Create a validation function for transaction DataFrames

Input schema (from schemas/transaction.py):
- txn_id: StringType, nullable=False
- account_id: StringType, nullable=False
- amount: DecimalType(18, 2), nullable=False
- currency: StringType, nullable=False
- timestamp: TimestampType, nullable=False
```

### Example: Providing Business Context

**Without Business Rules:**
```
> Validate the amount field
```

**With Business Rules:**
```
> Validate transaction amounts according to banking rules:
- Amount must be positive (> 0)
- Amount must not exceed $10,000 (daily transaction limit)
- Amount must use exactly 2 decimal places
- Reject any amount with NULL or negative values
```

---

## Format: Output Structure

### Specify How You Want the Output

**Unspecified:**
```
> Create a transaction validator
```
*Result: Claude might create a class, function, or full module*

**Specified:**
```
> Create a transaction validator as a Python function

Signature:
def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    '''
    Returns: (valid_df, invalid_df)
    '''

Include:
- Docstring with type hints
- Validation logic using PySpark DataFrame operations
- Logging for each validation rule
- Return tuple of (valid_df, invalid_df)
```

### Common Output Formats for Data Engineering

| Format | Use Case | Example |
|--------|----------|---------|
| **Function** | Reusable transformation | `def aggregate_daily_totals(df): ...` |
| **Class** | Complex pipeline | `class PaymentProcessor: ...` |
| **pytest Test** | Test cases | `def test_validate_positive_amounts(): ...` |
| **PySpark DataFrame** | Data transformation | `df.filter(...).groupBy(...)` |
| **StructType** | Schema definition | `StructType([StructField(...)])` |
| **Markdown** | Documentation | Table, list, code blocks |
| **JSON** | Configuration | `{"validation_rules": [...]}` |

---

## Data Engineering Examples

### Example 1: Generate a Validation Function

#### ❌ Weak Prompt

```
> Create validation
```

**Problems:**
- No clear task (validate what?)
- No context (schema, rules)
- No format (function? class?)

#### ✅ Strong Prompt

```
> Generate a PySpark validation function for banking transactions

TASK: Create function to validate transaction data quality

CONTEXT:
Input schema (defined in schemas/transaction.py):
```python
StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("currency", StringType(), nullable=False)
])
```

Validation rules:
- txn_id must be unique (no duplicates)
- amount must be positive (> 0)
- amount must not exceed $10,000
- currency must be in ['USD', 'EUR', 'GBP']

FORMAT:
Return as Python function:
```python
def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """Validate banking transactions.

    Returns:
        Tuple of (valid_df, invalid_df)
    """
```

Include:
- Type hints
- Docstring
- PySpark DataFrame operations (no pandas/collect)
- Logging for validation failures
```

**Claude's Response:**

```python
from pyspark.sql import DataFrame
from pyspark.sql import functions as F
from pyspark.sql.window import Window
from typing import Tuple
import logging

logger = logging.getLogger(__name__)


def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """Validate banking transactions for data quality and business rules.

    Validation rules:
    - txn_id must be unique (no duplicates)
    - amount must be positive (> 0)
    - amount must not exceed $10,000
    - currency must be in ['USD', 'EUR', 'GBP']

    Args:
        df: Input DataFrame with transaction schema

    Returns:
        Tuple of (valid_df, invalid_df) where:
        - valid_df: Transactions passing all validation rules
        - invalid_df: Transactions failing one or more rules
    """
    # Add validation columns
    validated_df = df.withColumn(
        "is_unique",
        F.count("*").over(Window.partitionBy("txn_id")) == 1
    ).withColumn(
        "is_positive_amount",
        F.col("amount") > 0
    ).withColumn(
        "is_within_limit",
        F.col("amount") <= 10000
    ).withColumn(
        "is_valid_currency",
        F.col("currency").isin(["USD", "EUR", "GBP"])
    )

    # Separate valid and invalid
    valid_df = validated_df.filter(
        F.col("is_unique") &
        F.col("is_positive_amount") &
        F.col("is_within_limit") &
        F.col("is_valid_currency")
    ).drop("is_unique", "is_positive_amount", "is_within_limit", "is_valid_currency")

    invalid_df = validated_df.filter(
        ~F.col("is_unique") |
        ~F.col("is_positive_amount") |
        ~F.col("is_within_limit") |
        ~F.col("is_valid_currency")
    )

    # Log validation results
    valid_count = valid_df.count()
    invalid_count = invalid_df.count()
    total_count = df.count()

    logger.info(f"Validation complete: {valid_count}/{total_count} valid ({valid_count/total_count*100:.1f}%)")
    if invalid_count > 0:
        logger.warning(f"{invalid_count} transactions failed validation")

    return (valid_df, invalid_df)
```

**Why This Works:**
- ✅ Follows exact function signature requested
- ✅ Implements all validation rules
- ✅ Uses PySpark DataFrame operations (no collect)
- ✅ Includes comprehensive docstring with type hints
- ✅ Adds logging as requested
- ✅ Returns tuple format as specified

### Example 2: Generate PySpark Aggregation

#### ❌ Weak Prompt

```
> Calculate daily totals
```

#### ✅ Strong Prompt

```
> Generate a PySpark aggregation to calculate daily transaction totals by account

TASK: Aggregate transactions to daily account-level totals

CONTEXT:
Input DataFrame schema:
- txn_id: StringType
- account_id: StringType
- amount: DecimalType(18, 2)
- currency: StringType
- timestamp: TimestampType

Business logic:
- Group by account_id and date (extract from timestamp)
- Sum amounts for each account per day
- Include transaction count per account per day
- Filter to USD currency only
- Sort by date descending, then amount descending

FORMAT:
Return as PySpark transformation code that:
1. Extracts date from timestamp
2. Filters for USD
3. Groups and aggregates
4. Sorts results

Output schema:
- account_id: StringType
- date: DateType
- total_amount: DecimalType(18, 2)
- txn_count: IntegerType
```

**Claude's Response:**

```python
from pyspark.sql import DataFrame
from pyspark.sql import functions as F

def aggregate_daily_totals(df: DataFrame) -> DataFrame:
    """Aggregate transactions to daily account-level totals.

    Args:
        df: Input DataFrame with transaction data

    Returns:
        DataFrame with daily totals per account
    """
    result_df = (
        df
        # Extract date from timestamp
        .withColumn("date", F.to_date(F.col("timestamp")))

        # Filter to USD only
        .filter(F.col("currency") == "USD")

        # Group and aggregate
        .groupBy("account_id", "date")
        .agg(
            F.sum("amount").alias("total_amount"),
            F.count("*").alias("txn_count")
        )

        # Sort by date desc, then amount desc
        .orderBy(F.col("date").desc(), F.col("total_amount").desc())
    )

    return result_df
```

---

## Practice Exercises

### 📝 Exercise 1: Transaction Deduplication

**Scenario:** You need to remove duplicate transactions from a DataFrame.

**Your Task:** Write a prompt to generate a deduplication function.

**Requirements:**
- Remove duplicates based on txn_id
- Keep the earliest transaction (by timestamp)
- Log how many duplicates were removed
- Return deduplicated DataFrame

**Try crafting your prompt using Task + Context + Format before looking at the solution.**

---

### ✅ Solution: Exercise 1

**Effective Prompt:**

```
> Generate a PySpark function to deduplicate transactions

TASK: Remove duplicate transactions, keeping the earliest by timestamp

CONTEXT:
Input DataFrame schema:
```python
StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("timestamp", TimestampType(), nullable=False)
])
```

Deduplication logic:
- Group by txn_id
- Keep transaction with earliest timestamp
- Log number of duplicates removed

FORMAT:
Python function with signature:
```python
def deduplicate_transactions(df: DataFrame) -> DataFrame:
    """Remove duplicate transactions."""
```

Include:
- Type hints and docstring
- Window function to partition by txn_id
- Logging statement with duplicate count
```

**Claude's Response:**

```python
from pyspark.sql import DataFrame
from pyspark.sql import functions as F
from pyspark.sql.window import Window
import logging

logger = logging.getLogger(__name__)


def deduplicate_transactions(df: DataFrame) -> DataFrame:
    """Remove duplicate transactions, keeping earliest by timestamp.

    Args:
        df: Input DataFrame with transaction data

    Returns:
        DataFrame with duplicates removed
    """
    # Count original records
    original_count = df.count()

    # Add row number partitioned by txn_id, ordered by timestamp
    window_spec = Window.partitionBy("txn_id").orderBy("timestamp")

    deduplicated_df = (
        df
        .withColumn("row_num", F.row_number().over(window_spec))
        .filter(F.col("row_num") == 1)
        .drop("row_num")
    )

    # Log deduplication results
    final_count = deduplicated_df.count()
    duplicates_removed = original_count - final_count

    logger.info(f"Deduplication complete: removed {duplicates_removed} duplicates")
    logger.info(f"Original: {original_count}, Final: {final_count}")

    return deduplicated_df
```

---

### 📝 Exercise 2: Currency Conversion

**Scenario:** Convert transaction amounts from various currencies to USD.

**Your Task:** Write a prompt to generate a currency conversion function.

**Requirements:**
- Support EUR, GBP → USD conversion
- Use exchange rates: EUR=1.10, GBP=1.25
- Add new column `amount_usd` (keep original amount)
- Handle unknown currencies (leave as NULL with warning)

**Try it yourself first!**

---

### ✅ Solution: Exercise 2

**Effective Prompt:**

```
> Generate a PySpark function to convert transaction amounts to USD

TASK: Add USD-converted amount column to transactions

CONTEXT:
Input schema:
- amount: DecimalType(18, 2)
- currency: StringType (values: USD, EUR, GBP)

Conversion rates (to USD):
- USD: 1.00 (no conversion)
- EUR: 1.10
- GBP: 1.25
- Unknown: NULL (log warning)

FORMAT:
Function signature:
```python
def convert_to_usd(df: DataFrame) -> DataFrame:
    """Add amount_usd column with USD-converted amounts."""
```

Output:
- Add new column: amount_usd (DecimalType(18, 2))
- Keep original amount and currency columns
- Log warning for unknown currencies
```

**Claude's Response:**

```python
from pyspark.sql import DataFrame
from pyspark.sql import functions as F
from decimal import Decimal
import logging

logger = logging.getLogger(__name__)


def convert_to_usd(df: DataFrame) -> DataFrame:
    """Add USD-converted amount column to transactions.

    Conversion rates:
    - USD: 1.00
    - EUR: 1.10
    - GBP: 1.25

    Args:
        df: Input DataFrame with amount and currency columns

    Returns:
        DataFrame with additional amount_usd column
    """
    # Define conversion rates
    conversion_rates = {
        "USD": Decimal("1.00"),
        "EUR": Decimal("1.10"),
        "GBP": Decimal("1.25")
    }

    # Create mapping expression
    conversion_expr = F.when(F.col("currency") == "USD", F.col("amount") * 1.00)
    for currency, rate in conversion_rates.items():
        if currency != "USD":
            conversion_expr = conversion_expr.when(
                F.col("currency") == currency,
                F.col("amount") * float(rate)
            )
    conversion_expr = conversion_expr.otherwise(F.lit(None))

    # Add amount_usd column
    result_df = df.withColumn("amount_usd", conversion_expr)

    # Log warnings for unknown currencies
    unknown_currencies = (
        result_df
        .filter(F.col("amount_usd").isNull())
        .select("currency")
        .distinct()
        .collect()
    )

    if unknown_currencies:
        logger.warning(f"Unknown currencies found: {[row.currency for row in unknown_currencies]}")

    return result_df
```

---

## Common Mistakes

### Mistake 1: Too Vague

❌ **Bad:**
```
> Create a data pipeline
```

✅ **Good:**
```
> Create a PySpark ETL pipeline to:
1. Read transactions from S3 (parquet format)
2. Validate using validation rules in config/rules.yaml
3. Transform to daily aggregations
4. Write to Delta Lake table: banking.daily_transaction_totals
```

### Mistake 2: Missing Context

❌ **Bad:**
```
> Fix the error
```

✅ **Good:**
```
> Fix the AnalysisException in pipelines/aggregations/daily_totals.py line 45

Error message:
"Column 'transaction_date' does not exist. Available columns: txn_id, amount, timestamp"

The code tries to group by 'transaction_date' but the column is named 'timestamp'.
Please fix by extracting date from timestamp column.
```

### Mistake 3: Undefined Output

❌ **Bad:**
```
> Generate tests
```

✅ **Good:**
```
> Generate pytest tests for validate_transactions() function

Test cases needed:
1. test_valid_transactions_pass_all_rules()
2. test_negative_amount_rejected()
3. test_amount_exceeds_limit_rejected()
4. test_invalid_currency_rejected()
5. test_duplicate_txn_id_rejected()

Use pyspark.testing.utils.assertDataFrameEqual for assertions
Create sample DataFrames with 3-5 records per test
```

---

## Summary

In this subsection, you learned:

### The Golden Formula
- ✅ **TASK**: Clear action verb + specific scope
- ✅ **CONTEXT**: Schema, business rules, technology stack
- ✅ **FORMAT**: Function signature, output structure, requirements

### Key Principles
- ✅ Start with a clear action verb
- ✅ Provide complete context (schema, rules, examples)
- ✅ Specify exact output format
- ✅ Reference existing patterns when available

### Data Engineering Applications
- ✅ Validation functions with business rules
- ✅ Aggregation logic with PySpark
- ✅ Transformation pipelines
- ✅ Test case generation

---

## Next Steps

👉 **[Continue to 1.4.3: Being Clear and Direct](./04-03-being-clear-direct.md)**

**Practice more:**
- Try the formula on your own data pipelines
- Experiment with different levels of context
- Compare outputs with varying specificity

---

**Related Sections:**
- [Phase 1.4.1: Tutorial How-To](./04-01-tutorial-how-to.md) - Tutorial basics
- [Phase 1.4.3: Being Clear and Direct](./04-03-being-clear-direct.md) - Next topic
- [Phase 1.3.3: Memory & Context](./03-03-memory-context.md) - CLAUDE.md for context

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 01_Basic_Prompt_Structure.ipynb
