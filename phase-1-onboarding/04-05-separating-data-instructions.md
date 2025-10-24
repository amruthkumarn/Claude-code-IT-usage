# Phase 1.4.5: Separating Data and Instructions

**Learning Objectives:**
- Understand why separating data from instructions matters
- Master XML tags for clear content boundaries
- Use delimiters effectively in prompts
- Apply separation techniques to data engineering tasks

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.4 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how separating data from instructions prevents misinterpretation.

### Try This Right Now

```bash
claude
```

**In the Claude REPL, compare these two prompts:**

**❌ WITHOUT Separation (Ambiguous):**
```
> Validate this transaction data:
> TXN001, ACC123, 100.50, USD
> Amount must be positive
>
> Generate validation function
```

**✅ WITH Separation (Clear):**
```
> Generate a PySpark validation function

<sample_data>
TXN001, ACC123, 100.50, USD
</sample_data>

<validation_rules>
- Amount must be positive
- Currency must be in ['USD', 'EUR', 'GBP']
</validation_rules>

Task: Create validate_transactions() function using these rules
```

**Exit Claude:**
```bash
Ctrl+D
```

**Compare:** The second prompt clearly separates sample data from validation rules and task instructions!

**Key Insight:** XML tags create clear boundaries, preventing Claude from confusing data with instructions.

---

## Table of Contents
1. [Why Separation Matters](#why-separation-matters)
2. [Using XML Tags](#using-xml-tags)
3. [Delimiter Strategies](#delimiter-strategies)
4. [Banking Data Engineering Examples](#banking-data-engineering-examples)
5. [Practice Exercises](#practice-exercises)
6. [Summary](#summary)

---

## Why Separation Matters

### The Confusion Problem

When data and instructions mix, Claude can misinterpret your intent.

**❌ Problem Example:**

```
> Create a validation function
> Test with: TXN001, amount -50.00
> Amount must be positive
> Return valid and invalid DataFrames
```

**What's unclear:**
- Is "Amount must be positive" part of test data description or a validation rule?
- Is "Return valid and invalid DataFrames" test data or an instruction?
- Should "-50.00" be validated or is it example of invalid data?

**✅ Solution with Separation:**

```
> Create a PySpark transaction validation function

<validation_rules>
- Amount must be positive (> 0)
- Amount must not exceed $10,000
</validation_rules>

<test_data>
TXN001, ACC123, -50.00, USD  # Invalid: negative amount
TXN002, ACC456, 100.50, USD  # Valid
</test_data>

<output_format>
Return: Tuple of (valid_df, invalid_df)
</output_format>

Implementation requirements:
- Use PySpark DataFrame operations
- Include type hints and docstrings
```

**Result:** Crystal clear what each section means!

---

## Using XML Tags

### Why XML Tags Work

XML tags provide **visual and semantic boundaries** that help Claude understand structure:

```
<tag_name>
Content that belongs together
</tag_name>
```

**Benefits:**
- ✅ Visually distinct
- ✅ Hierarchical structure
- ✅ Familiar format (like HTML)
- ✅ Clear start and end boundaries

### Common XML Tags for Data Engineering

| Tag | Use For | Example |
|-----|---------|---------|
| `<schema>` | DataFrame schemas | StructType definitions |
| `<sample_data>` | Example data | CSV rows, JSON objects |
| `<query>` | SQL queries | SELECT statements |
| `<rules>` | Business logic | Validation rules |
| `<config>` | Configuration | Spark settings, file paths |
| `<error>` | Error messages | Stack traces, exceptions |
| `<expected_output>` | Desired results | Expected DataFrame content |

### Example: Transaction Validation

```
> Generate a PySpark validation function for banking transactions

<schema>
StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("currency", StringType(), nullable=False),
    StructField("timestamp", TimestampType(), nullable=False)
])
</schema>

<validation_rules>
1. Schema validation:
   - All fields must match schema types
   - No null values allowed in non-nullable fields

2. Business rules:
   - amount must be positive (> 0)
   - amount must not exceed $10,000 (daily limit)
   - currency must be in ['USD', 'EUR', 'GBP']
   - txn_id must match pattern: TXN[0-9]{6}

3. Compliance (PCI-DSS):
   - No CVV field present
   - No full card number (16-digit sequences)
</validation_rules>

<sample_data>
# Valid transaction
{"txn_id": "TXN001", "account_id": "ACC123", "amount": 100.50, "currency": "USD", "timestamp": "2024-01-15T10:30:00"}

# Invalid: negative amount
{"txn_id": "TXN002", "account_id": "ACC456", "amount": -50.00, "currency": "USD", "timestamp": "2024-01-15T11:00:00"}

# Invalid: over limit
{"txn_id": "TXN003", "account_id": "ACC789", "amount": 15000.00, "currency": "USD", "timestamp": "2024-01-15T12:00:00"}
</sample_data>

<output_format>
Function signature:
def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame, Dict[str, int]]:
    """
    Returns:
        valid_df: Transactions passing all validations
        invalid_df: Transactions failing validations (with error_reason column)
        stats: Dict with validation counts per rule
    """
</output_format>

Implementation requirements:
- Use PySpark DataFrame operations (no pandas/collect)
- Log validation failures (mask account_id for PCI-DSS)
- Include pytest test cases using sample_data
- File: pipelines/validators/transaction_validator.py
```

**Why This Works:**
- ✅ Schema is clearly separated from rules
- ✅ Sample data is distinct from expected output
- ✅ Validation rules are grouped logically
- ✅ No ambiguity about what's data vs. instruction

---

## Delimiter Strategies

### Beyond XML: Other Separation Techniques

#### 1. Markdown Code Blocks

**Use for:** Code snippets, data samples, schemas

```
> Debug this PySpark pipeline

Current code:
```python
def process_transactions(df: DataFrame) -> DataFrame:
    return df.filter(col("amount") > 0)
```

Error message:
```
AnalysisException: Column 'amount' does not exist
Available columns: ['txn_id', 'account_id', 'transaction_amount']
```

Fix: Update column name from 'amount' to 'transaction_amount'
```

#### 2. Section Headers

**Use for:** Long prompts with multiple parts

```
> Generate a complete ETL pipeline

## Input Source
- Format: Parquet
- Location: s3://banking/raw/transactions/
- Schema: transaction_schema (defined in schemas/transaction.py)

## Transformation Logic
1. Deduplicate on txn_id (keep earliest by timestamp)
2. Validate amounts (positive, under $10K)
3. Enrich with customer_dim (broadcast join)
4. Calculate running_balance (window function)

## Output Destination
- Format: Delta Lake
- Location: delta_lake.processed_transactions
- Partition by: txn_date
- Mode: append

## Requirements
- Add error handling for missing files
- Log processing metrics (record counts, duration)
- Include pytest tests with sample data
```

#### 3. Triple-Quote Strings

**Use for:** SQL queries, configuration files

```
> Optimize this query

Query to optimize:
"""
SELECT
    account_id,
    SUM(amount) as total_amount,
    COUNT(*) as txn_count
FROM transactions
WHERE timestamp >= '2024-01-01'
GROUP BY account_id
HAVING SUM(amount) > 10000
ORDER BY total_amount DESC
"""

Current performance: 45 minutes on 100M rows
Target: Under 15 minutes

Cluster: 4 executors, 8GB each
```

#### 4. Bullet Lists

**Use for:** Validation rules, requirements, test cases

```
> Create validation logic

Validation rules:
• Schema: All fields must match transaction_schema
• Business: Amount between $0.01 and $10,000
• Compliance: No PII in logs (mask account_id)
• Quality: No duplicate txn_id

Test cases:
• Valid transaction: TXN001, amount 100.50
• Invalid: TXN002, amount -50.00 (negative)
• Invalid: TXN003, amount 15000.00 (over limit)
• Invalid: TXN004, duplicate txn_id
```

---

## Banking Data Engineering Examples

### Example 1: Schema Evolution Request

**❌ Without Separation:**

```
> Add a merchant_category field to transactions
> Should be string, nullable
> Update the schema and add validation
> merchant_category in ['retail', 'dining', 'travel', 'other']
> Existing transactions should get 'other' as default
```

**Problems:**
- Schema details mixed with validation rules
- Migration logic unclear
- Hard to distinguish requirements from implementation notes

**✅ With Separation:**

```
> Implement schema evolution for transactions table

<current_schema>
StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("currency", StringType(), nullable=False),
    StructField("timestamp", TimestampType(), nullable=False)
])
</current_schema>

<new_field>
StructField("merchant_category", StringType(), nullable=True)
</new_field>

<validation_rules>
merchant_category must be in ['retail', 'dining', 'travel', 'other']
</validation_rules>

<migration_logic>
For existing transactions:
- Set merchant_category = 'other' (default)
- Preserve all existing fields
- Maintain backward compatibility
</migration_logic>

<compliance_requirements>
- Audit log all schema changes (SOX requirement)
- Test rollback procedure
- Document schema version change (v1.0 → v1.1)
</compliance_requirements>

Tasks:
1. Update schema in schemas/transaction.py
2. Create migration script with Delta Lake MERGE
3. Update validation function to check merchant_category
4. Add pytest tests for new field validation
5. Update documentation

File: migrations/add_merchant_category_v1_1.py
```

---

### Example 2: Complex Query with Multiple Data Sources

**❌ Without Separation:**

```
> Write a query to join transactions with customers and merchants
> transactions has txn_id, account_id, amount, merchant_id
> customers has account_id, customer_name, risk_score
> merchants has merchant_id, merchant_name, category
> Calculate total amount per customer and merchant category
> Filter to high-risk customers (risk_score > 0.7)
> Order by total amount descending
```

**Problems:**
- Three schemas mixed together
- Business logic scattered
- Output format unclear

**✅ With Separation:**

```
> Create a PySpark aggregation joining multiple tables

<table_schemas>
transactions:
- txn_id: StringType
- account_id: StringType
- amount: DecimalType(18, 2)
- merchant_id: StringType
- timestamp: TimestampType

customers:
- account_id: StringType (PK)
- customer_name: StringType
- risk_score: DoubleType (range: 0.0 to 1.0)

merchants:
- merchant_id: StringType (PK)
- merchant_name: StringType
- category: StringType ['retail', 'dining', 'travel', 'other']
</table_schemas>

<join_logic>
transactions LEFT JOIN customers ON account_id
transactions LEFT JOIN merchants ON merchant_id
</join_logic>

<business_logic>
1. Filter: risk_score > 0.7 (high-risk customers only)
2. Group by: customer_name, merchant category
3. Aggregate: SUM(amount) as total_amount, COUNT(*) as txn_count
4. Sort: total_amount DESC
</business_logic>

<performance_requirements>
- Use broadcast join for merchants table (< 10K rows)
- Partition by timestamp for efficient filtering
- Target execution time: < 5 minutes on 10M transactions
</performance_requirements>

<output_schema>
- customer_name: StringType
- merchant_category: StringType
- total_amount: DecimalType(18, 2)
- txn_count: IntegerType
- risk_score: DoubleType
</output_schema>

Generate PySpark code with proper join optimization
File: pipelines/analytics/high_risk_spending_analysis.py
```

---

### Example 3: Debugging with Error Context

**❌ Without Separation:**

```
> The pipeline is failing with AnalysisException: Column 'txn_date' does not exist
> The code does df.groupBy('txn_date').agg(sum('amount'))
> But the schema shows timestamp field, not txn_date
> How do I fix this?
```

**Problems:**
- Error message inline with explanation
- Code snippet not clearly marked
- Schema information vague

**✅ With Separation:**

```
> Debug column name mismatch in aggregation pipeline

<file_location>
pipelines/aggregations/daily_summary.py
Line 67-70
</file_location>

<current_code>
```python
def calculate_daily_totals(df: DataFrame) -> DataFrame:
    return df.groupBy('txn_date') \
             .agg(sum('amount').alias('daily_total'))
```
</current_code>

<error_message>
```
pyspark.sql.utils.AnalysisException: Column 'txn_date' does not exist.
Available columns: ['txn_id', 'account_id', 'amount', 'currency', 'timestamp']
```
</error_message>

<schema_reference>
Defined in: schemas/transaction.py

StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("timestamp", TimestampType(), nullable=False),  # ← Date info is here
    ...
])
</schema_reference>

<diagnosis>
The code tries to group by 'txn_date' but:
1. Schema has 'timestamp' (TimestampType), not 'txn_date'
2. Need to extract date from timestamp first
</diagnosis>

Fix required:
1. Extract date from timestamp column: F.to_date(F.col("timestamp")).alias("txn_date")
2. Then group by the new txn_date column
3. Update code to use extracted date

Provide corrected code with the fix applied
```

---

## Practice Exercises

### 📝 Exercise 1: Add XML Tags to Improve Clarity

**Scenario:** This prompt mixes data with instructions. Add XML tags to separate them.

**Original Prompt:**
```
> Create a fraud detection function
> Check if transaction amount is more than 3x the account's 30-day average
> Sample account history: ACC123 has amounts [100, 150, 120, 110, 130]
> Current transaction: 600
> Should flag as potential fraud
> Also check for velocity: more than 5 transactions in 10 minutes
> Return boolean is_fraud and fraud_reason
```

**Your Task:** Rewrite with proper XML tag separation.

---

### ✅ Solution: Exercise 1

**Improved Prompt:**

```
> Create a PySpark fraud detection function

<fraud_detection_rules>
1. Amount anomaly:
   - Flag if transaction amount > 3x account's 30-day average

2. Velocity check:
   - Flag if account has >5 transactions within 10-minute window
</fraud_detection_rules>

<sample_data>
Account: ACC123
30-day transaction history:
[100.00, 150.00, 120.00, 110.00, 130.00]

30-day average: 122.00
3x threshold: 366.00

Current transaction: 600.00  # Should flag: 600 > 366
</sample_data>

<output_format>
Return: Tuple of (is_fraud: bool, fraud_reason: str)

Examples:
- (True, "Amount exceeds 3x average")
- (True, "Velocity check failed: 6 transactions in 8 minutes")
- (False, "No fraud indicators")
</output_format>

<implementation_requirements>
- Use PySpark window functions for 30-day average calculation
- Use window functions for velocity check (last 10 minutes)
- Function signature: detect_fraud(df: DataFrame, txn: Row) -> Tuple[bool, str]
- Include pytest tests with sample_data
</implementation_requirements>

File: pipelines/ml/fraud_detector.py
```

**Why This Works:**
- ✅ Fraud rules clearly separated from sample data
- ✅ Expected output format is distinct
- ✅ Implementation requirements separate from business logic
- ✅ No ambiguity about what's data vs. instruction

---

### 📝 Exercise 2: Structure a Complex Query Request

**Scenario:** You need to write a prompt for a complex multi-table aggregation. Structure it with proper separation.

**Requirements:**
- Join transactions, customers, and merchant_categories tables
- Filter to last 30 days
- Calculate total spending per customer per category
- Include customer risk scores
- Output as DataFrame

**Your Task:** Write a well-structured prompt using XML tags and sections.

---

### ✅ Solution: Exercise 2

**Effective Prompt:**

```
> Create a PySpark aggregation for customer spending analysis

<table_schemas>
## transactions
- txn_id: StringType, PK
- account_id: StringType, FK → customers
- merchant_id: StringType, FK → merchants
- amount: DecimalType(18, 2)
- timestamp: TimestampType
- status: StringType ['completed', 'pending', 'failed']

## customers
- account_id: StringType, PK
- customer_name: StringType
- risk_score: DoubleType (0.0 to 1.0)
- customer_since: DateType

## merchants
- merchant_id: StringType, PK
- merchant_name: StringType
- category: StringType ['retail', 'dining', 'travel', 'other']
</table_schemas>

<business_logic>
1. Time filter: Last 30 days (timestamp >= current_date - 30)
2. Status filter: Only 'completed' transactions
3. Joins:
   - transactions → customers (on account_id)
   - transactions → merchants (on merchant_id)
4. Aggregation:
   - Group by: account_id, customer_name, merchant category, risk_score
   - Sum: total_amount = SUM(amount)
   - Count: txn_count = COUNT(*)
5. Sort: total_amount DESC
</business_logic>

<performance_optimization>
- Broadcast join for merchants table (< 100K rows)
- Partition by timestamp date for filter pushdown
- Limit to top 1000 customers by spending
</performance_optimization>

<output_schema>
- account_id: StringType
- customer_name: StringType
- risk_score: DoubleType
- merchant_category: StringType
- total_amount: DecimalType(18, 2)
- txn_count: IntegerType
- first_txn_date: DateType
- last_txn_date: DateType
</output_schema>

<compliance_notes>
- Mask customer_name in any logs (GDPR)
- Audit query execution (SOX requirement)
- Do not materialize full result (data minimization)
</compliance_notes>

Generate PySpark code with:
- Function signature: analyze_customer_spending(spark: SparkSession) -> DataFrame
- Type hints and comprehensive docstring
- Error handling for missing tables
- Logging for execution metrics

File: pipelines/analytics/customer_spending_analysis.py
```

**Why This Works:**
- ✅ All three schemas clearly separated
- ✅ Business logic distinct from performance considerations
- ✅ Output schema explicitly defined
- ✅ Compliance requirements separated
- ✅ Implementation details at the end

---

### 📝 Exercise 3: Separate Test Data from Test Logic

**Scenario:** Create a prompt for generating pytest tests with clear separation.

**Your Task:** Write a prompt that separates test cases from test logic requirements.

---

### ✅ Solution: Exercise 3

**Effective Prompt:**

```
> Generate pytest tests for transaction validation function

<function_under_test>
File: pipelines/validators/transaction_validator.py

def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """
    Validates transaction data against business rules.

    Returns:
        Tuple of (valid_df, invalid_df)
    """
</function_under_test>

<validation_rules_tested>
1. Amount must be positive (> 0)
2. Amount must not exceed $10,000
3. Currency must be in ['USD', 'EUR', 'GBP']
4. txn_id must be unique (no duplicates)
</validation_rules_tested>

<test_cases>
## Test Case 1: All Valid Transactions
Input:
- TXN001, ACC123, 100.50, USD
- TXN002, ACC456, 250.00, EUR
Result: All records in valid_df, invalid_df empty

## Test Case 2: Negative Amount (Rule 1)
Input:
- TXN003, ACC789, -50.00, USD
Result: Record in invalid_df with error "negative_amount"

## Test Case 3: Amount Over Limit (Rule 2)
Input:
- TXN004, ACC123, 15000.00, USD
Result: Record in invalid_df with error "exceeds_limit"

## Test Case 4: Invalid Currency (Rule 3)
Input:
- TXN005, ACC456, 100.00, JPY
Result: Record in invalid_df with error "invalid_currency"

## Test Case 5: Duplicate Transaction (Rule 4)
Input:
- TXN006, ACC123, 100.00, USD
- TXN006, ACC456, 200.00, USD
Result: Duplicate in invalid_df with error "duplicate_txn_id"

## Test Case 6: Multiple Validation Failures
Input:
- TXN007, ACC789, -100.00, JPY
Result: In invalid_df with errors ["negative_amount", "invalid_currency"]
</test_cases>

<test_requirements>
- Use pytest framework
- Use pyspark.testing.utils.assertDataFrameEqual for assertions
- Create SparkSession fixture
- Follow Arrange-Act-Assert pattern
- Use descriptive test function names (test_rejects_negative_amounts)
- Include docstrings explaining what each test validates
- Use chispa or test data builders for sample DataFrames
</test_requirements>

<test_file_structure>
File: tests/test_transaction_validator.py

Structure:
1. Imports and fixtures
2. Helper function: create_transaction_df(transactions: List[Dict])
3. Test functions (one per test case)
4. Each test:
   - Arrange: Create input DataFrame
   - Act: Call validate_transactions()
   - Assert: Check valid_df and invalid_df contents
</test_file_structure>

Generate complete pytest test file with all 6 test cases
```

**Why This Works:**
- ✅ Function being tested clearly identified
- ✅ Validation rules separate from test cases
- ✅ Each test case has clear input and expected output
- ✅ Test requirements distinct from test data
- ✅ File structure guidance separate from test logic

---

## 💡 Try This: Practice XML Tag Usage

Test different separation strategies:

```bash
claude
```

**In the Claude REPL, try these progressively better prompts:**

```
# Level 1: No separation
> Create a function to calculate daily transaction totals
> Use transactions with txn_id, amount, timestamp
> Group by date from timestamp
> Return DataFrame with date and total_amount

# Level 2: Markdown code blocks
> Create a function to calculate daily transaction totals
>
> Schema:
> ```
> txn_id, amount, timestamp
> ```
>
> Logic: Group by date extracted from timestamp, sum amounts

# Level 3: XML tags (best!)
> Create a function to calculate daily transaction totals
>
> <schema>
> StructType([
>     StructField("txn_id", StringType()),
>     StructField("amount", DecimalType(18, 2)),
>     StructField("timestamp", TimestampType())
> ])
> </schema>
>
> <aggregation_logic>
> 1. Extract date from timestamp: to_date(col("timestamp"))
> 2. Group by date
> 3. Sum amounts per date
> </aggregation_logic>
>
> <output_schema>
> - txn_date: DateType
> - total_amount: DecimalType(18, 2)
> </output_schema>
```

**Exit Claude:**
```bash
Ctrl+D
```

**Compare outputs:** Notice how XML tags lead to better structured, more accurate code!

---

## Common Mistakes

### Mistake 1: Mixing Sample Data with Requirements

**❌ Bad:**
```
> Validate transactions
> TXN001 should pass because amount is positive
> TXN002 should fail because amount is negative
> Make sure amounts are positive
```

**What's wrong:** Hard to tell if "TXN001 should pass" is sample data or a validation rule.

**✅ Good:**
```
> Validate transactions

<validation_rule>
Amounts must be positive (> 0)
</validation_rule>

<test_expectations>
TXN001 (amount: 100.00) → PASS
TXN002 (amount: -50.00) → FAIL with "negative_amount" error
</test_expectations>
```

---

### Mistake 2: Using Too Many Nested Tags

**❌ Over-complicated:**
```
<data>
  <schema>
    <fields>
      <field>
        <name>txn_id</name>
        <type>StringType</type>
      </field>
    </fields>
  </schema>
</data>
```

**✅ Keep it simple:**
```
<schema>
txn_id: StringType
amount: DecimalType(18, 2)
timestamp: TimestampType
</schema>
```

---

### Mistake 3: Forgetting to Close Tags

**❌ Broken:**
```
<schema>
StructType([...])

<validation_rules>
Amount must be positive
```

**✅ Always close:**
```
<schema>
StructType([...])
</schema>

<validation_rules>
Amount must be positive
</validation_rules>
```

---

## Summary

In this subsection, you learned:

### Why Separation Matters
- ✅ Prevents Claude from confusing data with instructions
- ✅ Makes prompts easier to understand and maintain
- ✅ Reduces ambiguity and misinterpretation
- ✅ Improves output quality and accuracy

### XML Tag Techniques
- ✅ Use `<schema>`, `<sample_data>`, `<rules>`, `<output_format>` tags
- ✅ Create clear visual and semantic boundaries
- ✅ Keep tag names descriptive and consistent
- ✅ Don't over-nest tags - keep it simple

### Other Separation Strategies
- ✅ Markdown code blocks for code snippets
- ✅ Section headers for long prompts
- ✅ Triple-quote strings for SQL queries
- ✅ Bullet lists for rules and requirements

### Banking Data Engineering Applications
- ✅ Separate schemas from validation rules
- ✅ Distinguish test data from test logic
- ✅ Separate business rules from technical requirements
- ✅ Clear boundaries between data sources in joins
- ✅ Separate compliance requirements from functionality

---

## Key Takeaways

**Golden Rule:** If you're providing both data and instructions, use XML tags or delimiters to separate them.

**Best Practices:**
1. **Schema = `<schema>` tag** - Always wrap schema definitions
2. **Data = `<sample_data>` tag** - Separate examples from rules
3. **Rules = `<validation_rules>` or `<business_logic>` tag** - Keep logic distinct
4. **Output = `<output_format>` tag** - Specify expected results separately

**When to Use XML Tags:**
- ✅ Providing sample data or test cases
- ✅ Defining schemas or data structures
- ✅ Specifying validation or business rules
- ✅ Showing error messages or logs
- ✅ Multiple data sources (joins, merges)

---

## Next Steps

👉 **[Continue to 1.4.6: Formatting Output](./04-06-formatting-output.md)**

**Practice before proceeding:**
1. ✅ Review your recent prompts - where could XML tags help?
2. ✅ Try the "Try This" exercises with your own data pipelines
3. ✅ Experiment with different tag names
4. ✅ Test how separation affects output quality

---

**Related Sections:**
- [Phase 1.4.2: Basic Prompt Structure](./04-02-basic-prompt-structure.md) - Task + Context + Format
- [Phase 1.4.3: Being Clear and Direct](./04-03-being-clear-direct.md) - Clarity principles
- [Phase 1.4.6: Formatting Output](./04-06-formatting-output.md) - Next topic

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 04_Separating_Data_and_Instructions.ipynb
