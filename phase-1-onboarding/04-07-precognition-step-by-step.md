# Phase 1.4.7: Precognition (Thinking Step-by-Step)

**Learning Objectives:**
- Use chain of thought prompting for complex problems
- Get Claude to reason through data engineering challenges
- Apply step-by-step thinking to debugging and design
- Improve accuracy through structured reasoning

**Time Commitment:** 45 minutes

**Prerequisites:** Phase 1.4.1-1.4.6 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how step-by-step reasoning improves Claude's analysis.

```bash
claude
```

**In the Claude REPL, compare:**

**❌ WITHOUT Step-by-Step:**
```
> Why is my PySpark aggregation taking 45 minutes?
```

**✅ WITH Step-by-Step:**
```
> Analyze why my PySpark aggregation is slow

Think through this step-by-step:
1. First, identify potential bottlenecks (shuffle, skew, memory)
2. Then, analyze the join strategy being used
3. Next, evaluate the partition count
4. Finally, provide top 3 optimization recommendations with estimated impact

<code>
df.groupBy('account_id', 'date').agg(sum('amount'))
</code>

<context>
- Data: 10M transactions
- Cluster: 4 executors, 8GB each
- Current time: 45 minutes
</context>
```

**Exit Claude:**
```bash
Ctrl+D
```

**Compare:** Step-by-step prompt forces systematic analysis!

**Key Insight:** Breaking complex problems into steps leads to better, more thorough solutions.

---

## Table of Contents
1. [What is Precognition?](#what-is-precognition)
2. [Chain of Thought Prompting](#chain-of-thought-prompting)
3. [Step-by-Step Reasoning](#step-by-step-reasoning)
4. [Banking Data Engineering Examples](#banking-data-engineering-examples)
5. [Practice Exercises](#practice-exercises)
6. [Summary](#summary)

---

## What is Precognition?

### Definition

**Precognition** = Asking Claude to think through a problem step-by-step BEFORE providing the final answer.

**Why it works:**
- ✅ Forces systematic analysis
- ✅ Reduces rushing to conclusions
- ✅ Catches edge cases
- ✅ Produces more accurate results
- ✅ Shows reasoning process

### The Problem with Direct Answers

**Direct prompt:**
```
> Debug this pipeline error
```

**Result:** Claude might guess the issue without thorough analysis.

**Step-by-step prompt:**
```
> Debug this pipeline error

Analyze step-by-step:
1. First, examine the error message and stack trace
2. Then, identify which line of code is failing
3. Next, determine what data state would cause this error
4. Finally, propose a fix with explanation
```

**Result:** Systematic debugging process that finds the root cause.

---

## Chain of Thought Prompting

### Basic Pattern

```
> Solve [PROBLEM]

Think through this step-by-step:
1. [Step 1 description]
2. [Step 2 description]
3. [Step 3 description]
...
N. [Final step description]
```

### Example: Data Quality Issue

**Prompt:**
```
> We're seeing duplicate transactions in our aggregated daily totals

Investigate this step-by-step:
1. First, identify where duplicates could be introduced
2. Then, check if the deduplication logic is working
3. Next, verify the group-by keys are correct
4. Finally, propose a fix to prevent duplicates

<pipeline_code>
```python
daily_totals = (
    transactions_df
    .groupBy('account_id', 'date')
    .agg(sum('amount').alias('total'))
)
```
</pipeline_code>

<symptoms>
- Expected: 1 row per account per date
- Actual: Some accounts have 2-3 rows for same date
- Impact: Daily totals are inflated by 2-3x
</symptoms>
```

**Claude's Step-by-Step Response:**

```
Step 1: Identifying where duplicates could be introduced
---
Possible sources:
- Upstream data has duplicate transactions (same txn_id appears multiple times)
- The 'date' column has inconsistent values (different timestamps converting to different dates)
- Multiple pipeline runs are writing to the same output without deduplication

Step 2: Checking deduplication logic
---
Current code has NO deduplication step before aggregation. The groupBy only groups by
account_id and date, but if the source transactions_df has duplicates, they will
all be summed together.

Missing: No .dropDuplicates(['txn_id']) before aggregation

Step 3: Verifying group-by keys
---
The groupBy uses 'date' field, but I don't see where 'date' is extracted from 'timestamp'.
If transactions_df has a 'timestamp' field, the code needs:
  .withColumn('date', to_date(col('timestamp')))

If the date extraction happens elsewhere and is inconsistent, same txn_id could
appear on different dates.

Step 4: Proposed fix
---
```python
from pyspark.sql.functions import to_date, col, sum

daily_totals = (
    transactions_df
    # Add: Extract consistent date from timestamp
    .withColumn('date', to_date(col('timestamp')))

    # Add: Remove duplicate transactions
    .dropDuplicates(['txn_id'])

    # Now aggregate
    .groupBy('account_id', 'date')
    .agg(sum('amount').alias('total'))
)
```

Root cause: Missing deduplication before aggregation
Impact: 100% fix - eliminates duplicate counting
Testing: Compare before/after counts per account
```

**Why This Works:** Claude systematically analyzes each potential issue before proposing solution.

---

## Step-by-Step Reasoning

### Technique 1: Numbered Steps

**Pattern:**
```
Approach this step-by-step:
1. [First, do X]
2. [Then, do Y]
3. [Next, do Z]
4. [Finally, do W]
```

**Example:**
```
> Design a PySpark pipeline for fraud detection

Approach step-by-step:
1. First, define what features indicate fraud
2. Then, design window functions to calculate those features
3. Next, define threshold rules for flagging
4. Finally, provide the complete PySpark code

<requirements>
- Detect velocity fraud (>5 txns in 10 minutes)
- Detect amount anomalies (>3x account average)
- Maintain 99.9% precision (low false positives)
</requirements>
```

---

### Technique 2: XML-Tagged Reasoning

**Pattern:**
```
Analyze this problem:

<step1_identify_issue>
[Claude describes the core issue]
</step1_identify_issue>

<step2_analyze_causes>
[Claude lists potential causes]
</step2_analyze_causes>

<step3_propose_solutions>
[Claude proposes solutions]
</step3_propose_solutions>

<step4_recommend_best>
[Claude recommends best approach with justification]
</step4_recommend_best>
```

**Example:**
```
> Our transaction processing pipeline is slow

Analyze using these steps:

<step1_identify_bottleneck>
Examine the execution plan and identify the slowest stage
</step1_identify_bottleneck>

<step2_diagnose_root_cause>
Determine why that stage is slow (shuffle, skew, memory, etc.)
</step2_diagnose_root_cause>

<step3_list_optimizations>
List 3-5 potential optimizations with estimated impact
</step3_list_optimizations>

<step4_prioritize_actions>
Rank optimizations by impact vs. effort
</step4_prioritize_actions>

<pipeline_code>
```python
result = (transactions
    .join(customers, 'account_id')
    .groupBy('account_id')
    .agg(sum('amount')))
```
</pipeline_code>
```

---

### Technique 3: Brainstorming Before Deciding

**Pattern:**
```
Before answering, first brainstorm:
- [List all possible approaches]
- [Consider pros/cons of each]
Then, recommend the best approach with justification
```

**Example:**
```
> How should we partition the transactions table in Delta Lake?

Before deciding, brainstorm:

<possible_partition_strategies>
List 3-4 partition strategies with pros/cons:
1. By date
2. By account_id
3. By date AND account_id
4. By date AND region
</possible_partition_strategies>

<evaluation_criteria>
Evaluate each against:
- Query patterns (mostly by date range + account)
- Data volume (10M txns/day)
- Partition count target (100-1000 partitions)
- Maintenance overhead
</evaluation_criteria>

<recommendation>
Recommend best strategy with justification
</recommendation>

<query_patterns>
- 80% of queries: Filter by date range (7-30 days), then by account_id
- 15% of queries: Filter by date range only (aggregations)
- 5% of queries: Single account_id lookup (any date)
</query_patterns>
```

---

## Banking Data Engineering Examples

### Example 1: Root Cause Analysis for Data Quality Issue

**Prompt:**
```
> Investigate why 5% of transactions have null merchant_id

Perform root cause analysis step-by-step:

1. First, characterize the problem:
   - What percentage of nulls by date?
   - Are nulls clustered by specific accounts/merchants?
   - When did nulls first appear?

2. Then, trace the data flow:
   - Where does merchant_id originate?
   - What transformations touch merchant_id?
   - Could any transformation introduce nulls?

3. Next, form hypotheses:
   - List 3-5 potential root causes
   - For each, describe how to test the hypothesis

4. Finally, recommend investigation steps:
   - Prioritized list of SQL queries or code checks to run
   - Expected findings for each hypothesis

<context>
Pipeline: transaction_loader.py → transaction_enricher.py → daily_summary.py
merchant_id source: Kafka topic 'transactions' from payment gateway
Recent changes: Added new payment method (Apple Pay) last week
Null rate: 0% before last week, 5% since then
</context>
```

**Claude's Response:**
```
STEP 1: Characterize the Problem
================================
Based on the timeline, nulls started when Apple Pay was added. Let me characterize:

Temporal pattern:
- Pre-Apple Pay launch: 0% nulls
- Post-Apple Pay launch: 5% nulls
- This suggests the issue is related to the new payment method

I would run:
```sql
SELECT
  payment_method,
  COUNT(*) as total,
  COUNT(merchant_id) as non_null,
  (COUNT(*) - COUNT(merchant_id)) / COUNT(*) * 100 as null_percent
FROM transactions
WHERE date >= 'apple_pay_launch_date'
GROUP BY payment_method
```

Expected: Apple Pay transactions have ~100% null merchant_id, others ~0%

STEP 2: Trace Data Flow
========================
merchant_id data flow:
1. Payment gateway → Kafka topic 'transactions'
2. transaction_loader.py reads from Kafka
3. transaction_enricher.py joins with merchant_dim table

Hypothesis: Apple Pay transactions from payment gateway may not include merchant_id field,
OR the field is named differently for Apple Pay.

Validation check:
```python
# Check Kafka raw messages for Apple Pay txns
apple_pay_txns = (
    raw_kafka_df
    .filter(col('payment_method') == 'apple_pay')
    .select('txn_id', 'merchant_id', 'raw_payload')
)
# Examine raw_payload JSON structure
```

STEP 3: Form Hypotheses
========================

Hypothesis A: Apple Pay gateway doesn't send merchant_id
- Likelihood: HIGH
- Test: Examine raw Kafka messages for Apple Pay transactions
- Expected: merchant_id field missing from JSON payload

Hypothesis B: merchant_id field name is different for Apple Pay
- Likelihood: MEDIUM
- Test: Examine raw_payload for alternative fields (merchant_code, merchant_identifier)
- Expected: Different field name contains merchant info

Hypothesis C: transaction_enricher.py join fails for Apple Pay merchants
- Likelihood: LOW (would see nulls sporadically, not consistently for Apple Pay)
- Test: Check if Apple Pay merchant IDs exist in merchant_dim table
- Expected: All merchant IDs present

Hypothesis D: Schema mapping error in transaction_loader.py
- Likelihood: MEDIUM
- Test: Review schema mapping code for Apple Pay-specific logic
- Expected: merchant_id not mapped for apple_pay payment_method

STEP 4: Investigation Steps
============================

Priority 1: Check raw Kafka messages (Hypothesis A & B)
```python
from pyspark.sql.functions import get_json_object

kafka_messages = spark.read \
    .format('kafka') \
    .option('subscribe', 'transactions') \
    .load()

apple_pay_raw = (
    kafka_messages
    .filter(get_json_object(col('value'), '$.payment_method') == 'apple_pay')
    .withColumn('raw_json', col('value').cast('string'))
    .select('raw_json')
    .limit(10)
)
apple_pay_raw.show(truncate=False)
# Examine JSON structure for merchant_id field
```

Priority 2: Review transaction_loader.py schema mapping (Hypothesis D)
```python
# In transaction_loader.py, check if there's conditional logic:
if payment_method == 'apple_pay':
    # Is merchant_id mapping different here?
```

Priority 3: Check merchant_dim join (Hypothesis C)
```sql
SELECT
  COUNT(DISTINCT t.merchant_id) as unique_merchants_in_txns,
  COUNT(DISTINCT m.merchant_id) as unique_merchants_in_dim,
  COUNT(DISTINCT CASE WHEN m.merchant_id IS NULL THEN t.merchant_id END) as unmatched
FROM transactions t
LEFT JOIN merchant_dim m ON t.merchant_id = m.merchant_id
WHERE t.payment_method = 'apple_pay'
  AND t.date >= 'launch_date'
```

RECOMMENDED IMMEDIATE ACTION:
=============================
Most likely root cause: Apple Pay gateway JSON payload has a different structure
or uses a different field name for merchant information.

Immediate fix:
1. Examine 5-10 raw Kafka messages for Apple Pay transactions
2. Identify the correct field name for merchant info (might be 'apple_merchant_id' or 'tokenized_merchant')
3. Update transaction_loader.py to handle Apple Pay-specific field mapping:
   ```python
   .withColumn('merchant_id',
       when(col('payment_method') == 'apple_pay',
            get_json_object(col('value'), '$.apple_merchant_id'))
       .otherwise(get_json_object(col('value'), '$.merchant_id'))
   )
   ```
4. Backfill historical Apple Pay transactions with correct merchant_id
```

**Why This Works:**
- ✅ Systematic investigation from symptoms to root cause
- ✅ Multiple hypotheses considered
- ✅ Testable predictions for each hypothesis
- ✅ Prioritized investigation steps
- ✅ Immediate actionable recommendations

---

### Example 2: Pipeline Design with Step-by-Step

**Prompt:**
```
> Design a complete ETL pipeline for customer account balance tracking

Design step-by-step:

1. First, define the inputs and outputs:
   - Source tables and schemas
   - Target table and schema
   - Update frequency

2. Then, design the transformation logic:
   - How to calculate running balance
   - How to handle corrections and reversals
   - Window function approach

3. Next, plan for performance:
   - Partition strategy
   - Join optimizations
   - Caching decisions

4. Then, add data quality checks:
   - Validation rules
   - Reconciliation logic
   - Error handling

5. Finally, provide complete PySpark code

<requirements>
- Calculate end-of-day balance for each account
- Handle backdated corrections (adjust historical balances)
- Support both credits and debits
- Maintain 7-year history (SOX compliance)
- Process 10M transactions/day in under 30 minutes
</requirements>
```

---

## Practice Exercises

### 📝 Exercise 1: Debug with Step-by-Step

**Scenario:** A PySpark job is failing with "Out of Memory" error.

**Your Task:** Write a prompt using step-by-step reasoning to diagnose and fix the issue.

**Error:**
```
java.lang.OutOfMemoryError: GC overhead limit exceeded
  at org.apache.spark.util.collection.ExternalAppendOnlyMap.insertAll
```

---

### ✅ Solution: Exercise 1

**Effective Prompt:**
```
> Debug PySpark out-of-memory error

Diagnose step-by-step:

1. First, identify what operation is causing OOM:
   - Which transformation is triggering the error?
   - Is it during shuffle, collect, or broadcast?

2. Then, analyze the data volumes:
   - How much data is being processed?
   - Are there any data skew issues?
   - Is broadcast join being used on large tables?

3. Next, examine the executor configuration:
   - Executor memory allocation
   - Driver memory allocation
   - Number of executor cores

4. Finally, propose 3 fixes ranked by impact:
   - Code changes (use alternatives to collect, optimize joins)
   - Configuration changes (increase memory, adjust partitions)
   - Data changes (prune unnecessary columns, filter early)

<job_code>
```python
customers_df = spark.read.parquet('s3://data/customers')  # 10GB, 5M rows
transactions_df = spark.read.parquet('s3://data/transactions')  # 500GB, 100M rows

result = transactions_df.join(broadcast(customers_df), 'account_id') \
    .groupBy('account_id').agg(sum('amount'))

result.collect()  # ← Error occurs here
```
</job_code>

<cluster_config>
- Executors: 4
- Executor memory: 8GB
- Executor cores: 4
- Driver memory: 4GB
</cluster_config>
```

---

### 📝 Exercise 2: Design Decision with Reasoning

**Scenario:** Choose a partitioning strategy for a large transactions table.

**Your Task:** Write a prompt that makes Claude reason through the tradeoffs before recommending.

---

### ✅ Solution: Exercise 2

**Effective Prompt:**
```
> Choose optimal partitioning strategy for transactions table

Reason through this decision:

<step1_list_options>
List 4 possible partitioning strategies:
1. PARTITION BY (date)
2. PARTITION BY (account_id)
3. PARTITION BY (date, account_id)
4. PARTITION BY (date, region)
</step1_list_options>

<step2_evaluate_each>
For each strategy, evaluate:
- Query performance for common patterns
- Partition count (target: 100-1000)
- Write amplification
- Maintenance overhead
</step2_evaluate_each>

<step3_apply_context>
Apply our specific context:
- Query patterns: 70% by date range, 25% by account_id, 5% full scans
- Data volume: 10M transactions/day
- Retention: 7 years (25B total rows)
- Write pattern: Daily batch append at 2 AM
</step3_apply_context>

<step4_recommend>
Recommend best strategy with:
- Rationale for why this strategy wins
- Trade-offs being made
- Alternative if requirements change
</step4_recommend>
```

---

## Summary

In this subsection, you learned:

### Precognition Techniques
- ✅ Ask Claude to think step-by-step before answering
- ✅ Break complex problems into systematic stages
- ✅ Use XML tags to structure reasoning
- ✅ Force explicit brainstorming before conclusions

### Chain of Thought Benefits
- ✅ More accurate analysis and solutions
- ✅ Catches edge cases and nuances
- ✅ Shows reasoning process for verification
- ✅ Better debugging and root cause analysis

### Banking Data Engineering Applications
- ✅ Debugging complex pipeline failures
- ✅ Root cause analysis for data quality issues
- ✅ Performance optimization investigations
- ✅ Architecture and design decisions
- ✅ Multi-step ETL pipeline design

---

## Key Takeaways

**When to Use Step-by-Step:**
- Complex debugging (multiple potential causes)
- Design decisions (many trade-offs to consider)
- Root cause analysis (need systematic investigation)
- Performance optimization (multiple bottlenecks)
- Any problem where rushing to answer might miss important details

**Step-by-Step Pattern:**
```
> Solve [PROBLEM]

Approach step-by-step:
1. First, [identify/analyze/examine]...
2. Then, [investigate/evaluate/consider]...
3. Next, [compare/test/validate]...
4. Finally, [recommend/conclude/implement]...
```

**Remember:** Thinking out loud = Better answers!

---

## Next Steps

👉 **[Continue to 1.4.8: Using Examples (Few-Shot Prompting)](./04-08-using-examples.md)**

**Practice:**
1. ✅ Try debugging a recent issue with step-by-step reasoning
2. ✅ Design a new pipeline using the step-by-step approach
3. ✅ Compare direct vs. step-by-step prompts on same problem

---

**Related Sections:**
- [Phase 1.4.3: Being Clear and Direct](./04-03-being-clear-direct.md) - Clarity principles
- [Phase 1.4.9: Avoiding Hallucinations](./04-09-avoiding-hallucinations.md) - Verification
- [Phase 1.4.10: Complex Prompts](./04-10-complex-prompts.md) - Combining techniques

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 06_Precognition_Thinking_Step_by_Step.ipynb
