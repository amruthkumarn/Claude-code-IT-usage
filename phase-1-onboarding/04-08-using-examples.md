# Phase 1.4.8: Using Examples (Few-Shot Prompting)

**Learning Objectives:**
- Use examples to guide Claude's output format and style
- Master few-shot prompting for consistent results
- Apply examples to test generation and validation
- Understand when and how many examples to provide

**Time Commitment:** 45 minutes

**Prerequisites:** Phase 1.4.1-1.4.7 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how examples dramatically improve output consistency.

```bash
claude
```

**In the Claude REPL, compare:**

**❌ WITHOUT Examples:**
```
> Generate pytest test cases for transaction validation
```

**✅ WITH Examples:**
```
> Generate pytest test cases for transaction validation

Follow this pattern:

Example 1 - Valid transaction:
def test_accepts_valid_transaction():
    """Test that valid transactions pass validation."""
    # Arrange
    df = create_transaction_df([{"txn_id": "TXN001", "amount": 100.50}])
    # Act
    valid_df, invalid_df = validate_transactions(df)
    # Assert
    assert valid_df.count() == 1
    assert invalid_df.count() == 0

Example 2 - Invalid amount:
def test_rejects_negative_amount():
    """Test that negative amounts are rejected."""
    # Arrange
    df = create_transaction_df([{"txn_id": "TXN002", "amount": -50.00}])
    # Act
    valid_df, invalid_df = validate_transactions(df)
    # Assert
    assert valid_df.count() == 0
    assert invalid_df.count() == 1

Generate 3 more tests following this exact format
```

**Exit Claude:**
```bash
Ctrl+D
```

**Key Insight:** Examples show Claude exactly what you want - format, style, and structure!

---

## Table of Contents
1. [What is Few-Shot Prompting?](#what-is-few-shot-prompting)
2. [When to Use Examples](#when-to-use-examples)
3. [How Many Examples to Provide](#how-many-examples-to-provide)
4. [Banking Data Engineering Examples](#banking-data-engineering-examples)
5. [Practice Exercises](#practice-exercises)
6. [Summary](#summary)

---

## What is Few-Shot Prompting?

### Definition

**Few-shot prompting** = Providing examples of desired inputs/outputs to guide Claude's behavior.

**Terminology:**
- **Zero-shot**: No examples (just instructions)
- **One-shot**: One example
- **Few-shot**: 2-5 examples
- **Many-shot**: 6+ examples

### Why Examples Work

Examples are often MORE effective than descriptions because:
- ✅ Show exact format you want
- ✅ Demonstrate tone and style
- ✅ Clarify ambiguous requirements
- ✅ Provide concrete patterns to follow

**Without Example (description only):**
```
> Write docstrings in Google style with type hints
```

**With Example (showing the pattern):**
```
> Write docstrings following this pattern:

def calculate_balance(transactions: List[Transaction]) -> Decimal:
    """Calculate account balance from transaction history.

    Args:
        transactions: List of Transaction objects with amount and type fields

    Returns:
        Current account balance as Decimal

    Raises:
        ValueError: If transactions list is empty

    Example:
        >>> txns = [Transaction(100, 'credit'), Transaction(50, 'debit')]
        >>> calculate_balance(txns)
        Decimal('50.00')
    """
```

Claude will match this style automatically!

---

## When to Use Examples

### Use Case 1: Enforcing Output Format

**Scenario:** You want JSON with specific structure.

**Prompt:**
```
> Analyze transaction validation results and return as JSON

Example output format:
{
  "summary": {
    "total": 100,
    "valid": 97,
    "invalid": 3,
    "pass_rate": 97.0
  },
  "errors_by_type": {
    "negative_amount": 2,
    "invalid_currency": 1
  }
}

Analyze these results: [your data]
```

---

### Use Case 2: Test Case Generation

**Scenario:** Generate consistent pytest test structure.

**Prompt:**
```
> Generate pytest tests for amount validation function

Follow these examples:

# Example 1: Happy path
def test_accepts_valid_amount():
    """Valid amounts should pass validation."""
    # Arrange
    amount = Decimal("100.50")
    # Act
    result = validate_amount(amount)
    # Assert
    assert result.is_valid == True
    assert result.error is None

# Example 2: Edge case
def test_rejects_zero_amount():
    """Zero amounts should be rejected."""
    # Arrange
    amount = Decimal("0.00")
    # Act
    result = validate_amount(amount)
    # Assert
    assert result.is_valid == False
    assert result.error == "amount_must_be_positive"

Generate 4 more tests:
- Negative amount
- Amount over $10,000 limit
- Amount with >2 decimal places
- None/null amount
```

---

### Use Case 3: Code Review Comments

**Scenario:** Want consistent code review comment style.

**Prompt:**
```
> Review this PySpark code and provide feedback

Use this comment format:

Example 1:
**Issue:** Missing type hints
**Location:** Line 15, function `process_transactions`
**Impact:** Medium - Reduces code maintainability
**Fix:**
```python
def process_transactions(df: DataFrame) -> DataFrame:
    ...
```
**Rationale:** Type hints improve IDE support and catch errors early

Example 2:
**Issue:** Potential memory issue with collect()
**Location:** Line 42
**Impact:** High - Could cause OOM on large datasets
**Fix:**
```python
# Instead of:
total = df.select(sum('amount')).collect()[0][0]

# Use:
total = df.select(sum('amount')).first()[0]
```
**Rationale:** first() is more memory efficient than collect()

Now review this code: [paste code]
```

---

## How Many Examples to Provide

### General Guidelines

| Number of Examples | When to Use | Pros | Cons |
|-------------------|-------------|------|------|
| **0 (Zero-shot)** | Simple, well-defined tasks | Concise prompts | May miss nuances |
| **1 (One-shot)** | Clear format, single pattern | Quick, efficient | Limited context |
| **2-3 (Few-shot)** | **Most common - RECOMMENDED** | **Shows variation** | **Balanced** |
| **4-5 (Few-shot+)** | Complex patterns, edge cases | Comprehensive | Longer prompts |
| **6+ (Many-shot)** | Highly specific formatting | Very precise | Token-heavy |

### Rule of Thumb

**Start with 2-3 examples:**
- Example 1: Happy path (typical case)
- Example 2: Edge case or variation
- Example 3 (optional): Error case

---

## Banking Data Engineering Examples

### Example 1: Validation Rule Generation

**Prompt:**
```
> Generate PySpark validation rules for transaction fields

Follow these example patterns:

Example 1 - Range validation:
```python
def validate_amount(df: DataFrame) -> DataFrame:
    """Validate amount is within acceptable range."""
    return df.withColumn(
        "validation_error",
        when((col("amount") <= 0) | (col("amount") > 10000),
             "amount_out_of_range")
        .otherwise(None)
    )
```

Example 2 - Enum validation:
```python
def validate_currency(df: DataFrame) -> DataFrame:
    """Validate currency is in allowed list."""
    valid_currencies = ['USD', 'EUR', 'GBP']
    return df.withColumn(
        "validation_error",
        when(~col("currency").isin(valid_currencies),
             "invalid_currency")
        .otherwise(None)
    )
```

Example 3 - Pattern validation:
```python
def validate_txn_id(df: DataFrame) -> DataFrame:
    """Validate txn_id matches required pattern."""
    pattern = "^TXN[0-9]{6}$"
    return df.withColumn(
        "validation_error",
        when(~col("txn_id").rlike(pattern),
             "invalid_txn_id_format")
        .otherwise(None)
    )
```

Generate 3 more validators:
1. Account ID validation (pattern: ACC[0-9]{6})
2. Timestamp validation (not in future, not older than 1 year)
3. Merchant ID validation (not null, pattern: MERCH[A-Z0-9]{8})
```

---

### Example 2: Data Quality Test Patterns

**Prompt:**
```
> Generate data quality tests for transactions table

Use these patterns:

Example 1 - Completeness test:
```python
def test_no_null_in_required_fields():
    """Verify required fields have no null values."""
    null_counts = transactions_df.select([
        count(when(col(c).isNull(), c)).alias(c)
        for c in ['txn_id', 'account_id', 'amount']
    ]).collect()[0].asDict()

    for field, null_count in null_counts.items():
        assert null_count == 0, f"{field} has {null_count} null values"
```

Example 2 - Uniqueness test:
```python
def test_txn_id_is_unique():
    """Verify txn_id has no duplicates."""
    total_count = transactions_df.count()
    unique_count = transactions_df.select('txn_id').distinct().count()

    assert total_count == unique_count, \
        f"Found {total_count - unique_count} duplicate txn_ids"
```

Example 3 - Referential integrity test:
```python
def test_all_accounts_exist_in_customer_dim():
    """Verify all account_ids exist in customer dimension."""
    orphaned_accounts = (
        transactions_df
        .select('account_id')
        .distinct()
        .join(customers_df, 'account_id', 'left_anti')
    )

    count = orphaned_accounts.count()
    assert count == 0, f"Found {count} account_ids not in customer_dim"
```

Generate 5 more tests:
1. Validity: All amounts are positive
2. Consistency: Timestamp is within expected range
3. Accuracy: Currency codes match ISO 4217
4. Timeliness: No future timestamps
5. Range: Amounts don't exceed $1M (suspicious threshold)
```

---

### Example 3: Error Handling Patterns

**Prompt:**
```
> Add error handling to transaction processing functions

Follow these patterns:

Example 1 - Validation with custom exception:
```python
def validate_transaction_amount(amount: Decimal) -> None:
    """Validate transaction amount.

    Raises:
        ValidationError: If amount is invalid
    """
    if amount <= 0:
        raise ValidationError(
            error_code="INVALID_AMOUNT",
            message=f"Amount must be positive, got {amount}",
            field="amount",
            value=str(amount)
        )
    if amount > 10000:
        raise ValidationError(
            error_code="AMOUNT_EXCEEDS_LIMIT",
            message=f"Amount {amount} exceeds daily limit of $10,000",
            field="amount",
            value=str(amount)
        )
```

Example 2 - Retry logic for external calls:
```python
def fetch_exchange_rate(currency: str, date: datetime) -> Decimal:
    """Fetch exchange rate with retry logic."""
    max_retries = 3
    retry_delay = 1  # seconds

    for attempt in range(max_retries):
        try:
            return exchange_rate_api.get_rate(currency, date)
        except APIConnectionError as e:
            if attempt < max_retries - 1:
                logger.warning(f"API call failed, retry {attempt + 1}/{max_retries}")
                time.sleep(retry_delay * (2 ** attempt))  # Exponential backoff
            else:
                logger.error(f"API call failed after {max_retries} attempts")
                raise ExchangeRateUnavailableError(
                    f"Could not fetch rate for {currency} on {date}"
                ) from e
```

Example 3 - Graceful degradation:
```python
def enrich_with_merchant_data(df: DataFrame) -> DataFrame:
    """Enrich transactions with merchant data, handling missing merchants."""
    try:
        merchant_dim = spark.read.table("merchant_dim")
    except AnalysisException:
        logger.warning("merchant_dim table not found, skipping enrichment")
        return df.withColumn("merchant_name", lit(None)) \
                 .withColumn("merchant_category", lit("unknown"))

    return df.join(
        merchant_dim.select('merchant_id', 'merchant_name', 'merchant_category'),
        'merchant_id',
        'left'
    )
```

Add error handling to these functions:
1. read_transactions_from_kafka() - Handle connection failures
2. deduplicate_transactions() - Handle empty DataFrames
3. write_to_delta_lake() - Handle write conflicts
```

---

### Example 4: PySpark Transformation Patterns

**Prompt:**
```
> Generate PySpark transformations for transaction enrichment

Use these example patterns:

Example 1 - Window function for running balance:
```python
from pyspark.sql.window import Window

window_spec = Window.partitionBy('account_id') \
                    .orderBy('timestamp') \
                    .rowsBetween(Window.unboundedPreceding, Window.currentRow)

enriched_df = transactions_df.withColumn(
    'running_balance',
    sum('amount').over(window_spec)
)
```

Example 2 - Conditional column with business logic:
```python
enriched_df = transactions_df.withColumn(
    'risk_level',
    when(col('amount') > 5000, 'high')
    .when(col('amount') > 1000, 'medium')
    .otherwise('low')
)
```

Example 3 - Join with broadcast for small dimension:
```python
from pyspark.sql.functions import broadcast

enriched_df = transactions_df.join(
    broadcast(currency_rates_df),
    ['currency', 'date'],
    'left'
).withColumn(
    'amount_usd',
    col('amount') * col('exchange_rate')
)
```

Generate transformations for:
1. Add transaction age in days from timestamp
2. Categorize time of day (morning/afternoon/evening/night)
3. Add month-to-date transaction count per account
4. Flag duplicate transactions within 5-minute window
```

---

## Practice Exercises

### 📝 Exercise 1: Generate Validation Functions with Examples

**Your Task:** Write a prompt with 2-3 examples to generate schema validation functions.

**Requirements:**
- Check field types match schema
- Check nullable constraints
- Return validation errors

---

### ✅ Solution: Exercise 1

**Effective Prompt:**
```
> Generate schema validation functions for transaction fields

Follow these patterns:

Example 1 - Type validation:
```python
def validate_amount_type(df: DataFrame) -> DataFrame:
    """Validate amount is numeric type."""
    return df.withColumn(
        'type_error',
        when(~col('amount').cast('decimal').isNotNull(),
             'amount_not_numeric')
        .otherwise(None)
    )
```

Example 2 - Nullability validation:
```python
def validate_txn_id_not_null(df: DataFrame) -> DataFrame:
    """Validate txn_id is not null (required field)."""
    return df.withColumn(
        'null_error',
        when(col('txn_id').isNull(), 'txn_id_required')
        .otherwise(None)
    )
```

Generate validators for:
1. account_id: StringType, not null
2. currency: StringType, not null
3. timestamp: TimestampType, not null
4. merchant_id: StringType, nullable (allowed to be null)
```

---

### 📝 Exercise 2: Test Case Generation with Examples

**Your Task:** Create a prompt with examples for generating integration tests.

---

### ✅ Solution: Exercise 2

**Effective Prompt:**
```
> Generate integration tests for transaction ETL pipeline

Use this pattern:

Example 1 - End-to-end happy path:
```python
def test_pipeline_processes_valid_transactions():
    """Test complete pipeline with valid data."""
    # Arrange
    input_df = create_sample_transactions([
        {'txn_id': 'TXN001', 'amount': 100.50, 'currency': 'USD'}
    ])

    # Act
    result_df = run_transaction_pipeline(input_df)

    # Assert
    assert result_df.count() == 1
    assert result_df.filter(col('validation_status') == 'valid').count() == 1
    assert result_df.filter(col('enrichment_complete') == True).count() == 1
```

Example 2 - Error handling:
```python
def test_pipeline_handles_invalid_data():
    """Test pipeline correctly handles and logs invalid data."""
    # Arrange
    input_df = create_sample_transactions([
        {'txn_id': 'TXN002', 'amount': -50.00, 'currency': 'USD'}  # Invalid
    ])

    # Act
    result_df = run_transaction_pipeline(input_df)

    # Assert
    assert result_df.count() == 1
    assert result_df.filter(col('validation_status') == 'invalid').count() == 1
    assert result_df.filter(col('error_reason').contains('negative_amount')).count() == 1
```

Generate tests for:
1. Large batch processing (10K transactions)
2. Schema mismatch handling
3. Duplicate transaction detection
4. Performance within SLA (< 5 min for 100K records)
```

---

## Common Mistakes

### Mistake 1: Too Many Examples

**❌ Overkill (10 examples):**
```
Example 1: ...
Example 2: ...
...
Example 10: ...
```

**✅ Right amount (2-3 examples):**
```
Example 1: Happy path
Example 2: Edge case
Example 3: Error case
```

---

### Mistake 2: Examples Don't Match Task

**❌ Mismatched:**
```
> Generate PySpark validation code

Example: (shows pandas code instead of PySpark)
```

**✅ Matched:**
```
> Generate PySpark validation code

Example: (shows PySpark DataFrame operations)
```

---

### Mistake 3: Inconsistent Example Format

**❌ Inconsistent:**
```
Example 1:
def test_a(): ...

Example 2 - Different style:
class TestB:
    def test_method(): ...
```

**✅ Consistent:**
```
Example 1:
def test_valid_amount(): ...

Example 2:
def test_invalid_amount(): ...
```

---

## Summary

### Few-Shot Prompting Basics
- ✅ Provide 2-3 examples of desired output
- ✅ Examples show format, style, and structure
- ✅ More effective than descriptions alone
- ✅ Use for consistency across outputs

### When to Use Examples
- ✅ Enforcing specific output formats
- ✅ Generating consistent test cases
- ✅ Creating code with specific patterns
- ✅ Maintaining style guidelines

### Banking Applications
- ✅ Test case generation (pytest patterns)
- ✅ Validation function templates
- ✅ Error handling patterns
- ✅ PySpark transformation patterns
- ✅ Code review comment formats

---

## Key Takeaways

**Best Practices:**
1. **2-3 examples is optimal** for most tasks
2. **Show variety:** happy path, edge case, error case
3. **Match format:** Examples should look exactly like desired output
4. **Be consistent:** All examples should follow same structure

**Example Pattern:**
```
> Generate [TASK]

Follow this pattern:

Example 1 - [Happy path]:
[Show ideal case]

Example 2 - [Edge case]:
[Show variation or boundary]

Example 3 - [Error case]:
[Show error handling]

Now generate: [Your specific request]
```

---

## Next Steps

👉 **[Continue to 1.4.9: Avoiding Hallucinations](./04-09-avoiding-hallucinations.md)**

**Practice:**
1. ✅ Take a recent code generation task and add 2-3 examples
2. ✅ Compare output quality with vs without examples
3. ✅ Build a library of example patterns for your team

---

**Related Sections:**
- [Phase 1.4.6: Formatting Output](./04-06-formatting-output.md) - Output structure
- [Phase 1.4.2: Basic Prompt Structure](./04-02-basic-prompt-structure.md) - Format specification
- [Phase 1.4.10: Complex Prompts](./04-10-complex-prompts.md) - Combining techniques

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 07_Using_Examples_Few-Shot_Prompting.ipynb
