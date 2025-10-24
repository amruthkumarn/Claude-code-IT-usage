# Phase 1.4.10: Complex Prompts from Scratch

**Learning Objectives:**
- Combine all prompt engineering techniques
- Build complex prompts for end-to-end tasks
- Apply multi-step workflows to data engineering
- Master complete pipeline development prompts

**Time Commitment:** 45 minutes

**Prerequisites:** Phase 1.4.1-1.4.9 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how combining techniques creates powerful prompts.

**Complex Prompt Example:**

```bash
claude
```

**In the Claude REPL:**

```
> Build a complete transaction fraud detection pipeline

<role>
You are a senior data engineer specializing in PySpark-based fraud detection
for banking systems, with expertise in PCI-DSS compliance.
</role>

<task>
Design and implement a real-time fraud detection pipeline
</task>

<context>
Input: Kafka stream of transactions (10K/second)
Schema: txn_id, account_id, amount, merchant_id, timestamp
Cluster: 8 executors, 16GB each
Latency requirement: < 100ms per transaction
</context>

<fraud_rules>
1. Velocity: > 5 transactions within 10 minutes
2. Amount anomaly: > 3x account's 30-day average
3. Geographic: Transaction location > 500 miles from last transaction
</fraud_rules>

<output_format>
Provide in this order:
1. Architecture diagram (ASCII art)
2. PySpark Structured Streaming code
3. Window function specifications
4. Performance optimization notes
5. pytest test cases (3 examples)
</output_format>

<compliance>
- PCI-DSS: Mask account_id in fraud alerts
- SOX: Audit trail for all flagged transactions
- GDPR: Data retention (30 days for flagged, 7 days for clean)
</compliance>

Think step-by-step:
1. First, design the overall architecture
2. Then, implement the streaming query
3. Next, add fraud detection logic
4. Finally, add compliance and testing
```

**Exit Claude:**
```bash
Ctrl+D
```

**Key Insight:** Complex prompts combine Role + Context + Examples + Step-by-Step + Format!

---

## Table of Contents
1. [Building Complex Prompts](#building-complex-prompts)
2. [The Complete Prompt Template](#the-complete-prompt-template)
3. [Banking Data Engineering Examples](#banking-data-engineering-examples)
4. [Practice Exercises](#practice-exercises)
5. [Summary](#summary)

---

## Building Complex Prompts

### The Layers of a Complex Prompt

```
┌─────────────────────────────────────────┐
│ 1. ROLE (Who is Claude?)                │
│    Set expertise and perspective        │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 2. CONTEXT (What's the situation?)      │
│    Business requirements, constraints   │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 3. DATA/SCHEMA (What data exists?)      │
│    Input/output schemas, samples        │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 4. TASK (What needs to be done?)        │
│    Specific deliverable                 │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 5. EXAMPLES (What does good look like?) │
│    2-3 examples of desired output       │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 6. STEP-BY-STEP (How to approach it?)   │
│    Numbered reasoning steps             │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 7. OUTPUT FORMAT (How to structure?)    │
│    JSON, Markdown, code structure       │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ 8. VERIFICATION (How to test?)          │
│    Testing requirements, validation     │
└─────────────────────────────────────────┘
```

---

## The Complete Prompt Template

### Master Template for Complex Tasks

```
> [ONE-LINE TASK SUMMARY]

<role>
You are a [EXPERTISE LEVEL] [SPECIALTY] with [YEARS] experience in [DOMAIN].
Your focus is [PERSPECTIVE] and [KEY SKILLS].
</role>

<context>
Business context:
- [Business goal]
- [Key requirements]
- [Constraints]

Technical context:
- [Tech stack]
- [Data volumes]
- [Performance requirements]
- [Compliance needs]
</technical_context>

<input_data>
Schema:
```
[StructType definition or schema]
```

Sample data:
```
[Example records]
```
</input_data>

<task>
Primary deliverable: [What you need]

Sub-tasks:
1. [Subtask 1]
2. [Subtask 2]
3. [Subtask 3]
</task>

<examples>
Example 1 - [Type]:
[Show desired pattern]

Example 2 - [Type]:
[Show variation]
</examples>

<requirements>
Functional:
- [Requirement 1]
- [Requirement 2]

Non-functional:
- Performance: [SLA]
- Security: [Compliance]
- Maintainability: [Standards]
</requirements>

Think step-by-step:
1. First, [analyze/design/identify]...
2. Then, [implement/build/create]...
3. Next, [test/validate/verify]...
4. Finally, [document/deploy/deliver]...

<output_format>
Provide:
1. [Output component 1]
2. [Output component 2]
3. [Output component 3]

Format: [JSON/Markdown/Code]
</output_format>

<verification>
Testing requirements:
- [Test type 1]: [What to test]
- [Test type 2]: [What to test]

Success criteria:
- [Criterion 1]
- [Criterion 2]
</verification>
```

---

## Banking Data Engineering Examples

### Example 1: End-to-End ETL Pipeline

**Complete Prompt:**

```
> Build a production-grade daily transaction processing ETL pipeline

<role>
You are a principal data engineer with 10+ years building PySpark ETL pipelines
for banking transaction systems. You specialize in Delta Lake, data quality,
and regulatory compliance (PCI-DSS, SOX, GDPR).
</role>

<business_context>
Goal: Process daily transaction batches (T-1) and maintain 7-year history
Business impact: Powers daily reconciliation, fraud detection, regulatory reporting
SLA: Complete processing within 2-hour window (2 AM - 4 AM daily)
Data volume: 10M transactions/day, growing 20% annually
</business_context>

<technical_context>
Tech stack:
- PySpark 3.5+ on Databricks
- Delta Lake for storage
- AWS S3 for raw data (parquet)
- Confluent Schema Registry

Cluster:
- 8 executors × 16GB memory
- Auto-scaling enabled

Current state:
- Raw data arrives in s3://banking/raw/transactions/date=YYYY-MM-DD/
- Target: delta_lake.transactions (partitioned by txn_date)
</technical_context>

<input_schema>
Raw transaction schema (from payment gateway):
```python
StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("card_last_4", StringType(), nullable=True),
    StructField("amount", StringType(), nullable=False),  # String! Needs conversion
    StructField("currency", StringType(), nullable=False),
    StructField("merchant_id", StringType(), nullable=True),
    StructField("timestamp", StringType(), nullable=False),  # ISO 8601 string
    StructField("status", StringType(), nullable=False),
    StructField("payment_method", StringType(), nullable=False)
])
```

Target Delta Lake schema:
```python
StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("currency", StringType(), nullable=False),
    StructField("txn_date", DateType(), nullable=False),
    StructField("txn_timestamp", TimestampType(), nullable=False),
    StructField("merchant_id", StringType(), nullable=True),
    StructField("merchant_category", StringType(), nullable=True),  # Enriched
    StructField("status", StringType(), nullable=False),
    StructField("payment_method", StringType(), nullable=False),
    StructField("validation_status", StringType(), nullable=False),  # Added
    StructField("processed_timestamp", TimestampType(), nullable=False)  # Added
])
```
</input_schema>

<transformation_requirements>
1. Schema enforcement:
   - Convert amount from String to Decimal(18, 2)
   - Parse timestamp from ISO 8601 to TimestampType
   - Extract txn_date from timestamp

2. Validation:
   - Amount must be positive (> 0)
   - Amount must not exceed $1M (fraud threshold)
   - Currency must be in ['USD', 'EUR', 'GBP', 'CAD']
   - txn_id must be unique (no duplicates)
   - timestamp must not be in future

3. Enrichment:
   - Join with merchant_dim table to get merchant_category
   - Use broadcast join (merchant_dim is < 100MB)

4. Data quality:
   - Invalid records → quarantine table (delta_lake.transactions_quarantine)
   - Valid records → main table (delta_lake.transactions)
   - Generate data quality metrics
</transformation_requirements>

<compliance_requirements>
PCI-DSS:
- Do NOT store full card numbers (only last 4 digits allowed)
- Mask account_id in all logs
- Encrypt sensitive fields at rest (Delta Lake handles this)

SOX:
- Audit trail for all data modifications
- Track data lineage (source → processed)
- Retain all transactions for 7 years

GDPR:
- Support right to deletion (implement soft delete)
- Track data processing consent
</compliance_requirements>

<examples>
Example 1 - Schema conversion pattern:
```python
transformed_df = raw_df.withColumn(
    "amount",
    col("amount").cast(DecimalType(18, 2))
).withColumn(
    "txn_timestamp",
    to_timestamp(col("timestamp"), "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
).withColumn(
    "txn_date",
    to_date(col("txn_timestamp"))
)
```

Example 2 - Validation pattern:
```python
validated_df = transformed_df.withColumn(
    "validation_errors",
    array_remove(array(
        when(col("amount") <= 0, "negative_amount"),
        when(col("amount") > 1000000, "exceeds_threshold"),
        when(~col("currency").isin(['USD', 'EUR', 'GBP', 'CAD']), "invalid_currency")
    ), None)
)

valid_df = validated_df.filter(size(col("validation_errors")) == 0)
invalid_df = validated_df.filter(size(col("validation_errors")) > 0)
```

Example 3 - Delta Lake upsert pattern:
```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(spark, "delta_lake.transactions")

delta_table.alias("target").merge(
    valid_df.alias("source"),
    "target.txn_id = source.txn_id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
```
</examples>

Design and implement step-by-step:

1. First, design the overall pipeline architecture:
   - Data flow diagram (ASCII art)
   - Error handling strategy
   - Monitoring approach

2. Then, implement the core ETL logic:
   - Read from S3
   - Schema enforcement and type conversions
   - Validation logic
   - Enrichment with merchant data

3. Next, implement data quality and compliance:
   - Quarantine invalid records
   - Audit logging
   - Data quality metrics
   - Compliance checks

4. Then, implement Delta Lake writes:
   - Partition strategy
   - Merge logic for idempotency
   - Z-order optimization

5. Finally, add monitoring and testing:
   - Data quality alerts
   - Performance metrics
   - pytest integration tests

<output_format>
Provide complete implementation:

1. **Architecture Diagram** (ASCII art showing data flow)

2. **Main ETL Pipeline Code** (pipelines/etl/daily_transaction_processor.py)
   - Complete executable PySpark code
   - Type hints and comprehensive docstrings
   - Logging at key stages

3. **Configuration** (config/transaction_pipeline_config.yaml)
   - Configurable parameters (paths, thresholds, etc.)

4. **Data Quality Metrics** (JSON format)
   - Metrics to track and alert on

5. **Testing** (tests/test_daily_transaction_processor.py)
   - 5 pytest test cases covering:
     - Happy path (valid transactions)
     - Invalid data handling
     - Duplicate detection
     - Schema enforcement
     - Delta Lake merge correctness

6. **Deployment Guide** (Markdown)
   - How to deploy and run
   - Monitoring setup
   - Troubleshooting common issues
</output_format>

<success_criteria>
Pipeline must:
- ✅ Process 10M transactions in < 2 hours
- ✅ Achieve 99.9% data quality (< 0.1% quarantine rate)
- ✅ Pass all PCI-DSS compliance checks
- ✅ Support idempotent re-runs (same input = same output)
- ✅ Generate data quality metrics automatically
- ✅ Have >= 80% test coverage
</success_criteria>
```

---

### Example 2: Real-Time Fraud Detection System

**Complete Prompt:**

```
> Design and implement a real-time fraud detection system using PySpark Structured Streaming

<role>
You are a senior ML engineer specializing in real-time fraud detection for banking,
with expertise in PySpark Structured Streaming, Kafka, and low-latency systems.
</role>

<business_context>
Goal: Detect fraudulent transactions in real-time (< 100ms latency)
Business impact: Prevent fraud losses ($10M+ annually), improve customer trust
Volume: 10,000 transactions/second peak, 2M transactions/day average
False positive tolerance: < 1% (to avoid customer friction)
</business_context>

<technical_context>
Architecture:
- Input: Kafka topic 'transactions' (Avro format)
- Output: Kafka topic 'fraud-alerts' (high-risk transactions)
- State store: RocksDB (for window aggregations)
- Cluster: 16 executors × 32GB memory

Latency requirements:
- P50: < 50ms
- P95: < 100ms
- P99: < 200ms
</technical_context>

<fraud_detection_rules>
Rule 1: Velocity Fraud
- Trigger: > 5 transactions from same account_id within 10 minutes
- Risk score: 0.7

Rule 2: Amount Anomaly
- Trigger: Transaction amount > 3× account's 30-day rolling average
- Risk score: 0.6

Rule 3: Geographic Anomaly
- Trigger: Transaction location > 500 miles from previous transaction AND < 1 hour apart
- Risk score: 0.8

Rule 4: Merchant Risk
- Trigger: Merchant is in high-risk category AND amount > $500
- Risk score: 0.5

Composite risk score:
- If any rule triggers: fraud_score = MAX(rule_scores)
- If multiple rules: fraud_score = MIN(1.0, SUM(rule_scores) × 0.8)
- Alert threshold: fraud_score >= 0.65
</fraud_detection_rules>

<schemas>
Input (Kafka 'transactions'):
```python
StructType([
    StructField("txn_id", StringType()),
    StructField("account_id", StringType()),
    StructField("amount", DecimalType(18, 2)),
    StructField("merchant_id", StringType()),
    StructField("location_lat", DoubleType()),
    StructField("location_lon", DoubleType()),
    StructField("timestamp", TimestampType())
])
```

Output (Kafka 'fraud-alerts'):
```python
{
  "txn_id": "string",
  "account_id": "string (masked)",
  "fraud_score": "double (0.0-1.0)",
  "triggered_rules": ["string array"],
  "risk_level": "low|medium|high|critical",
  "alert_timestamp": "timestamp"
}
```
</schemas>

Implement step-by-step:

1. Design the streaming architecture
2. Implement Kafka source/sink connectors
3. Implement each fraud detection rule with window functions
4. Combine rules into composite fraud score
5. Add performance optimizations (watermarks, state management)
6. Implement monitoring and alerting

<output_format>
Provide:
1. Streaming query code (PySpark)
2. Window function specifications
3. State management strategy
4. Performance tuning parameters
5. Monitoring queries (count alerts/minute, latency P95)
6. Test cases for each fraud rule
</output_format>
```

---

## Practice Exercises

### 📝 Exercise: Build Your Own Complex Prompt

**Scenario:** Create a complete prompt for a customer account balance tracker.

**Requirements:**
- Real-time balance updates
- Support corrections and reversals
- 7-year history (SOX)
- Handle high volume (100K accounts, 10M transactions/day)

**Your Task:** Build a complex prompt using all techniques from Phase 1.4.

---

## Summary

### Complex Prompt Components
- ✅ Role setting (expertise)
- ✅ Business + technical context
- ✅ Data schemas (input/output)
- ✅ Specific task breakdown
- ✅ Examples (2-3 patterns)
- ✅ Step-by-step approach
- ✅ Output format specification
- ✅ Verification/testing requirements

### When to Use Complex Prompts
- End-to-end system design
- Production pipeline implementation
- Multi-step workflows
- High-stakes compliance scenarios
- Complete feature development

---

## Key Takeaways

**Building Blocks Checklist:**

Before submitting a complex prompt, ensure you have:
- [ ] Clear role definition
- [ ] Business context and impact
- [ ] Complete schemas with examples
- [ ] Specific deliverables listed
- [ ] 2-3 examples of desired patterns
- [ ] Step-by-step instructions
- [ ] Output format specified
- [ ] Testing/verification requirements
- [ ] Compliance needs stated

**Remember:** Complex prompts are for complex tasks. For simple tasks, keep it simple!

---

## Next Steps

👉 **[Continue to 1.4.11: Chaining Prompts](./04-11-chaining-prompts.md)**

**Practice:**
1. ✅ Build a complex prompt for your current project
2. ✅ Test it and refine based on results
3. ✅ Save successful prompts as templates

---

**Related Sections:**
- [Phase 1.4.2: Basic Prompt Structure](./04-02-basic-prompt-structure.md) - Foundation
- [All Phase 1.4 Subsections](./04-00-prompt-engineering-index.md) - Techniques used

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 09_Complex_Prompts_from_Scratch.ipynb
