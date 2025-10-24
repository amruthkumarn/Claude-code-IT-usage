# Phase 1.4.6: Formatting Output

**Learning Objectives:**
- Control Claude's output structure and format
- Use JSON for structured data results
- Generate Markdown tables for reports
- Create compliance-ready documentation
- Apply output formatting to data engineering tasks

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.5 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how specifying output format dramatically improves usability.

### Try This Right Now

```bash
claude
```

**In the Claude REPL, compare these prompts:**

**❌ WITHOUT Format Specification:**
```
> Analyze the transaction validation results
```

**✅ WITH Format Specification:**
```
> Analyze transaction validation results

Output as JSON:
{
  "total_transactions": <count>,
  "valid_count": <count>,
  "invalid_count": <count>,
  "validation_errors": [
    {"error_type": "<type>", "count": <count>}
  ],
  "pass_rate_percent": <percentage>
}
```

**Exit Claude:**
```bash
Ctrl+D
```

**Compare:** The second prompt gives you JSON you can parse programmatically!

**Key Insight:** Specify the exact output format you need - Claude will match it!

---

## Table of Contents
1. [Why Output Format Matters](#why-output-format-matters)
2. [JSON Output](#json-output)
3. [Markdown Tables](#markdown-tables)
4. [Structured Reports](#structured-reports)
5. [Banking Data Engineering Examples](#banking-data-engineering-examples)
6. [Practice Exercises](#practice-exercises)
7. [Summary](#summary)

---

## Why Output Format Matters

### The Problem with Unstructured Output

**Without format specification, Claude might return:**
```
The validation found 3 errors. There were 2 negative amounts and 1 invalid currency.
Out of 100 transactions, 97 were valid and 3 were invalid, giving a pass rate of 97%.
```

**Problems:**
- ❌ Hard to parse programmatically
- ❌ Inconsistent structure
- ❌ Can't easily extract specific values
- ❌ Not suitable for automation

**With format specification:**
```json
{
  "total_transactions": 100,
  "valid_count": 97,
  "invalid_count": 3,
  "validation_errors": [
    {"error_type": "negative_amount", "count": 2},
    {"error_type": "invalid_currency", "count": 1}
  ],
  "pass_rate_percent": 97.0
}
```

**Benefits:**
- ✅ Easy to parse with `json.loads()`
- ✅ Consistent structure
- ✅ Extract specific values by key
- ✅ Perfect for automation and dashboards

---

## JSON Output

### When to Use JSON

**JSON is perfect for:**
- Validation results
- Test summaries
- Performance metrics
- Configuration files
- API responses
- Structured logs

### Basic JSON Formatting

**Prompt:**
```
> Validate transaction amounts and return results as JSON

<validation_rules>
- Amount must be positive (> 0)
- Amount must not exceed $10,000
</validation_rules>

<sample_data>
TXN001: 100.50 (valid)
TXN002: -50.00 (invalid: negative)
TXN003: 15000.00 (invalid: exceeds limit)
TXN004: 250.00 (valid)
</sample_data>

Output format:
```json
{
  "summary": {
    "total": <count>,
    "valid": <count>,
    "invalid": <count>
  },
  "validation_results": [
    {
      "txn_id": "<id>",
      "amount": <decimal>,
      "status": "valid|invalid",
      "errors": ["<error_type>"]
    }
  ]
}
```
```

**Claude's Response:**
```json
{
  "summary": {
    "total": 4,
    "valid": 2,
    "invalid": 2
  },
  "validation_results": [
    {
      "txn_id": "TXN001",
      "amount": 100.50,
      "status": "valid",
      "errors": []
    },
    {
      "txn_id": "TXN002",
      "amount": -50.00,
      "status": "invalid",
      "errors": ["negative_amount"]
    },
    {
      "txn_id": "TXN003",
      "amount": 15000.00,
      "status": "invalid",
      "errors": ["exceeds_limit"]
    },
    {
      "txn_id": "TXN004",
      "amount": 250.00,
      "status": "valid",
      "errors": []
    }
  ]
}
```

### Nested JSON Structures

For complex results, use nested JSON:

```
> Analyze daily transaction patterns

Output format:
```json
{
  "analysis_date": "YYYY-MM-DD",
  "daily_stats": {
    "total_transactions": <count>,
    "total_amount": <decimal>,
    "avg_transaction": <decimal>,
    "median_transaction": <decimal>
  },
  "by_currency": [
    {
      "currency": "<code>",
      "count": <count>,
      "total_amount": <decimal>
    }
  ],
  "by_status": {
    "completed": <count>,
    "pending": <count>,
    "failed": <count>
  },
  "anomalies": [
    {
      "type": "<anomaly_type>",
      "description": "<text>",
      "severity": "low|medium|high"
    }
  ]
}
```
```

---

## Markdown Tables

### When to Use Markdown Tables

**Tables are perfect for:**
- Data comparison reports
- Test results summary
- Schema documentation
- Data lineage reports
- Performance benchmarks
- Audit trails

### Basic Table Formatting

**Prompt:**
```
> Compare transaction processing performance across environments

<data>
Production: 500K txns/day, avg 2.3s latency, 99.9% success rate
Staging: 100K txns/day, avg 3.1s latency, 99.5% success rate
Dev: 10K txns/day, avg 5.2s latency, 98.8% success rate
</data>

Output as Markdown table with columns:
- Environment
- Daily Volume
- Avg Latency (s)
- Success Rate (%)
- Status (✅ Meets SLA / ⚠️ Warning / ❌ Below SLA)

SLA: < 3.0s latency, > 99.5% success rate
```

**Claude's Response:**

| Environment | Daily Volume | Avg Latency (s) | Success Rate (%) | Status |
|-------------|--------------|-----------------|------------------|---------|
| Production  | 500,000      | 2.3             | 99.9             | ✅ Meets SLA |
| Staging     | 100,000      | 3.1             | 99.5             | ⚠️ Warning (latency) |
| Dev         | 10,000       | 5.2             | 98.8             | ❌ Below SLA |

**SLA Criteria:** Latency < 3.0s, Success Rate > 99.5%

### Data Lineage Tables

**Prompt:**
```
> Document data lineage for customer_balance table

Output as Markdown table:

| Source Table | Join Key | Transformation | Data Added | PII Fields | Compliance |
|--------------|----------|----------------|------------|------------|------------|
```

**Claude's Response:**

| Source Table | Join Key | Transformation | Data Added | PII Fields | Compliance |
|--------------|----------|----------------|------------|------------|------------|
| transactions | account_id | SUM(amount) GROUP BY account_id, date | balance, txn_count | account_id (masked) | PCI-DSS Level 1 |
| customers | account_id | LEFT JOIN | customer_name, risk_score | customer_name, email | GDPR Article 6 |
| accounts | account_id | INNER JOIN | account_type, open_date | None | SOX Section 404 |
| audit_log | N/A | Record insertion | audit_timestamp, user_id | user_id (hashed) | SOX Section 302 |

---

## Structured Reports

### Compliance Audit Reports

**Prompt:**
```
> Generate a PCI-DSS compliance audit report for the payment processing pipeline

<audit_scope>
Pipeline: pipelines/payment_processor.py
Date: 2024-01-15
Auditor: Data Engineering Team
</audit_scope>

<checks_performed>
1. Check for CVV storage (PCI-DSS 3.2)
2. Check for full PAN exposure in logs (PCI-DSS 3.4)
3. Check for encryption at rest (PCI-DSS 3.4)
4. Check for data masking in non-prod environments (PCI-DSS 3.4)
5. Check for audit trail completeness (PCI-DSS 10.2)
</checks_performed>

Output format:
---
# PCI-DSS Compliance Audit Report

## Executive Summary
[Overall compliance status, critical findings]

## Audit Details
- Pipeline: [name]
- Date: [date]
- Scope: [what was audited]

## Findings

### ✅ Passed Checks
[List of passed checks with evidence]

### ❌ Failed Checks
[List of failed checks with severity, impact, remediation]

### ⚠️ Warnings
[List of warnings requiring attention]

## Compliance Score
[X/Y checks passed (Z%)]

## Recommendations
[Prioritized list of actions]

## Sign-Off
- Auditor: [name]
- Date: [date]
- Next Review: [date]
---
```

### Test Results Report

**Prompt:**
```
> Summarize pytest test results for transaction validation pipeline

<test_results>
tests/test_transaction_validator.py::test_valid_transactions PASSED
tests/test_transaction_validator.py::test_negative_amount_rejected PASSED
tests/test_transaction_validator.py::test_amount_exceeds_limit_rejected FAILED
tests/test_transaction_validator.py::test_invalid_currency_rejected PASSED
tests/test_transaction_validator.py::test_duplicate_txn_id_rejected PASSED
tests/test_transaction_validator.py::test_null_amount_rejected PASSED
</test_results>

Output as structured Markdown report:

---
# Test Report: Transaction Validator

## Summary
- Total Tests: X
- Passed: X (X%)
- Failed: X
- Skipped: X

## Status: ✅ PASS / ❌ FAIL

## Failed Tests
[Details of failures with error messages]

## Coverage
- Lines covered: X/Y (Z%)
- Branches covered: X/Y (Z%)

## Recommendations
[Suggested fixes for failures]
---
```

---

## Banking Data Engineering Examples

### Example 1: Data Quality Report

**Prompt:**
```
> Generate a data quality report for the transactions table

<quality_checks>
1. Completeness: Check for null values in required fields
2. Validity: Check if amounts are positive
3. Consistency: Check if timestamps are in valid range
4. Uniqueness: Check for duplicate txn_id
5. Timeliness: Check if data is within expected date range
</quality_checks>

<results>
- Completeness: 98.5% (1,500 nulls in merchant_id out of 100,000 rows)
- Validity: 99.2% (800 negative amounts)
- Consistency: 100%
- Uniqueness: 99.9% (100 duplicate txn_id)
- Timeliness: 95% (5,000 rows with future timestamps)
</results>

Output format:
```json
{
  "report_date": "YYYY-MM-DD",
  "table_name": "transactions",
  "total_rows": <count>,
  "quality_score": <percentage>,
  "dimensions": [
    {
      "dimension": "<name>",
      "score": <percentage>,
      "passed": <boolean>,
      "threshold": <percentage>,
      "issues_found": <count>,
      "severity": "low|medium|high|critical",
      "details": "<description>"
    }
  ],
  "critical_issues": [
    {
      "issue": "<description>",
      "impact": "<business_impact>",
      "remediation": "<action_items>"
    }
  ],
  "recommendations": [
    "<action_1>",
    "<action_2>"
  ]
}
```
```

**Claude's Response:**
```json
{
  "report_date": "2024-01-15",
  "table_name": "transactions",
  "total_rows": 100000,
  "quality_score": 98.5,
  "dimensions": [
    {
      "dimension": "Completeness",
      "score": 98.5,
      "passed": true,
      "threshold": 95.0,
      "issues_found": 1500,
      "severity": "low",
      "details": "1,500 null values in optional merchant_id field (1.5% of data)"
    },
    {
      "dimension": "Validity",
      "score": 99.2,
      "passed": true,
      "threshold": 99.0,
      "issues_found": 800,
      "severity": "medium",
      "details": "800 transactions with negative amounts (0.8% of data) - business rule violation"
    },
    {
      "dimension": "Consistency",
      "score": 100.0,
      "passed": true,
      "threshold": 99.5,
      "issues_found": 0,
      "severity": "low",
      "details": "All timestamps within valid range (2024-01-01 to 2024-01-15)"
    },
    {
      "dimension": "Uniqueness",
      "score": 99.9,
      "passed": true,
      "threshold": 99.9,
      "issues_found": 100,
      "severity": "high",
      "details": "100 duplicate txn_id values found - data integrity issue"
    },
    {
      "dimension": "Timeliness",
      "score": 95.0,
      "passed": true,
      "threshold": 95.0,
      "issues_found": 5000,
      "severity": "critical",
      "details": "5,000 transactions with future timestamps (5% of data) - system clock issue?"
    }
  ],
  "critical_issues": [
    {
      "issue": "Future timestamps detected",
      "impact": "Data may be invalid; reporting accuracy affected; compliance risk",
      "remediation": "1. Investigate source system clock drift 2. Quarantine affected records 3. Implement timestamp validation at ingestion"
    },
    {
      "issue": "Duplicate transaction IDs",
      "impact": "Double-counting in aggregations; reconciliation failures; audit trail gaps",
      "remediation": "1. Add unique constraint on txn_id 2. Implement deduplication logic 3. Investigate source system for duplicate generation"
    }
  ],
  "recommendations": [
    "Add real-time validation at data ingestion to catch future timestamps",
    "Implement unique constraint on txn_id column in source system",
    "Set up alerts for data quality scores falling below 99%",
    "Investigate negative amounts - may indicate refunds vs. validation errors",
    "Review merchant_id requirement - consider making field mandatory"
  ]
}
```

---

### Example 2: Performance Benchmark Table

**Prompt:**
```
> Compare PySpark join strategies for transaction enrichment

<benchmark_data>
Shuffle Join: 45 minutes, 2TB shuffle, 100% data processed
Broadcast Join: 12 minutes, 50GB broadcast, 100% data processed
Salted Join: 28 minutes, 1TB shuffle, 100% data processed
</benchmark_data>

<context>
Transactions: 500K rows, 2GB
Customer Dim: 2M rows, 8GB (too large for broadcast)
Cluster: 4 executors, 8GB memory each
</context>

Output as Markdown table with:
- Join Strategy
- Execution Time
- Shuffle Size
- Memory Usage
- Recommendation (✅/⚠️/❌)
- Notes

Include a "Winner" row highlighting the best approach
```

**Claude's Response:**

## PySpark Join Strategy Benchmark: Transaction Enrichment

**Context:** 500K transactions (2GB) joining with 2M customers (8GB) on 4 executors × 8GB

| Join Strategy | Execution Time | Shuffle Size | Memory Usage | Recommendation | Notes |
|---------------|----------------|--------------|--------------|----------------|--------|
| Shuffle Join  | 45 min         | 2TB          | 6GB/executor | ❌ Not Recommended | Default strategy - excessive shuffle |
| **Broadcast Join** | **12 min** | **50GB** | **7.5GB/executor** | ⚠️ **Winner (with caveat)** | **Requires customer_dim to fit in memory (~8GB broadcast)** |
| Salted Join   | 28 min         | 1TB          | 5GB/executor | ✅ Recommended Fallback | Reduces skew, more predictable |

### Recommendation

**Use Broadcast Join IF:**
- Customer dimension remains < 2M rows (< 8GB)
- Executors have sufficient memory (8GB+ per executor)
- Memory pressure is monitored

**Use Salted Join IF:**
- Customer dimension grows beyond broadcast threshold
- Memory is constrained
- Join key has skew (some accounts have many transactions)

### Performance Gain
- Broadcast Join: **73% faster** than Shuffle Join
- Salted Join: **38% faster** than Shuffle Join

---

### Example 3: Schema Change Impact Analysis

**Prompt:**
```
> Analyze impact of adding merchant_category field to transactions table

<proposed_change>
Add field: merchant_category (StringType, nullable=True)
Default value: 'other'
</proposed_change>

<downstream_dependencies>
1. daily_summary pipeline (reads transactions, aggregates by date)
2. fraud_detection pipeline (reads transactions, joins with merchants)
3. customer_spending_report (reads transactions, groups by account)
4. audit_trail_generator (reads transactions, writes to audit log)
5. real_time_alerts (streams transactions, filters by amount)
</downstream_dependencies>

Output format:

## Schema Change Impact Analysis

| Pipeline/Table | Impact Level | Breaking Change? | Action Required | Estimated Effort |
|----------------|--------------|------------------|-----------------|------------------|

### Impact Levels:
- 🟢 None: No changes needed
- 🟡 Low: Optional updates for new functionality
- 🟠 Medium: Updates recommended for full functionality
- 🔴 High: Breaking change - updates required

---
```

**Claude's Response:**

## Schema Change Impact Analysis
**Change:** Add `merchant_category` field (StringType, nullable=True) to `transactions` table

| Pipeline/Table | Impact Level | Breaking Change? | Action Required | Estimated Effort |
|----------------|--------------|------------------|-----------------|------------------|
| daily_summary  | 🟢 None | No | Optional: Add merchant_category to aggregation for enhanced reporting | 2 hours |
| fraud_detection | 🟡 Low | No | Optional: Use merchant_category for category-specific fraud rules | 4 hours |
| customer_spending_report | 🟡 Low | No | Recommended: Group by merchant_category for category breakdown | 3 hours |
| audit_trail_generator | 🟢 None | No | None - field will be automatically included in audit logs | 0 hours |
| real_time_alerts | 🟢 None | No | Optional: Add category-based alert rules (e.g., high-value travel) | 6 hours |

### Summary
- **Total Pipelines Analyzed:** 5
- **Breaking Changes:** 0
- **Recommended Updates:** 2 (customer_spending_report, real_time_alerts)
- **Optional Updates:** 2 (daily_summary, fraud_detection)
- **No Action Required:** 1 (audit_trail_generator)

### Deployment Plan
1. **Phase 1:** Add field with default value 'other' (backward compatible)
2. **Phase 2:** Backfill historical data (optional, for reporting accuracy)
3. **Phase 3:** Update customer_spending_report (highest value)
4. **Phase 4:** Update fraud_detection and real_time_alerts (as needed)

### Risk Assessment
- **Risk Level:** LOW
- **Backward Compatibility:** ✅ Maintained
- **Rollback Plan:** Drop column if needed (data loss acceptable if no critical dependencies)

---

## Practice Exercises

### 📝 Exercise 1: Format Validation Results as JSON

**Scenario:** Create a prompt that returns structured JSON validation results.

**Your Task:** Write a prompt for transaction validation that specifies exact JSON output format.

**Requirements:**
- Summary section (totals, pass rate)
- Detailed results per validation rule
- List of invalid transactions with reasons

---

### ✅ Solution: Exercise 1

**Effective Prompt:**

```
> Validate transactions and return results as JSON

<validation_rules>
1. Amount must be positive (> 0)
2. Amount must not exceed $10,000
3. Currency must be in ['USD', 'EUR', 'GBP']
4. txn_id must be unique
</validation_rules>

<sample_data>
TXN001, ACC123, 100.50, USD
TXN002, ACC456, -50.00, USD
TXN003, ACC789, 15000.00, EUR
TXN004, ACC123, 200.00, JPY
TXN001, ACC999, 300.00, USD  (duplicate txn_id)
</sample_data>

Output format:
```json
{
  "validation_summary": {
    "total_transactions": <count>,
    "valid_count": <count>,
    "invalid_count": <count>,
    "pass_rate_percent": <percentage>
  },
  "rules_checked": [
    {
      "rule_id": "<id>",
      "rule_description": "<text>",
      "violations_count": <count>
    }
  ],
  "invalid_transactions": [
    {
      "txn_id": "<id>",
      "violations": ["<rule_id>"],
      "details": "<explanation>"
    }
  ]
}
```

Ensure all numbers are accurate based on the sample data
```

**Why This Works:**
- ✅ Explicit JSON structure specified
- ✅ Field names and types clear
- ✅ Nested objects for organization
- ✅ Can be parsed programmatically

---

### 📝 Exercise 2: Create a Pipeline Performance Report

**Scenario:** Format pipeline execution metrics as a Markdown table.

**Your Task:** Write a prompt that generates a performance comparison table.

**Data:**
- Pipeline A: 30 min, 100K records, $0.45 cost
- Pipeline B: 45 min, 100K records, $0.30 cost
- Pipeline C: 20 min, 100K records, $0.60 cost

---

### ✅ Solution: Exercise 2

**Effective Prompt:**

```
> Create a performance comparison report for ETL pipelines

<pipeline_data>
Pipeline A (transaction_loader):
- Execution time: 30 minutes
- Records processed: 100,000
- Cost: $0.45
- Success rate: 99.5%

Pipeline B (transaction_validator):
- Execution time: 45 minutes
- Records processed: 100,000
- Cost: $0.30
- Success rate: 99.8%

Pipeline C (transaction_enricher):
- Execution time: 20 minutes
- Records processed: 100,000
- Cost: $0.60
- Success rate: 99.2%
</pipeline_data>

<sla_targets>
- Execution time: < 30 minutes
- Cost: < $0.50
- Success rate: > 99.5%
</sla_targets>

Output as Markdown table with columns:
- Pipeline Name
- Execution Time
- Records Processed
- Cost ($)
- Success Rate (%)
- Cost per 1K Records
- SLA Status (✅ Met / ⚠️ Partial / ❌ Missed)

Include calculated fields and a summary row
```

**Why This Works:**
- ✅ All data provided with clear labels
- ✅ SLA targets specified
- ✅ Calculated fields requested (cost per 1K)
- ✅ Status indicators for quick assessment
- ✅ Markdown table format for easy sharing

---

### 📝 Exercise 3: Generate a Compliance Checklist

**Scenario:** Create a PCI-DSS compliance checklist in structured format.

**Your Task:** Write a prompt that generates a checklist for code review.

---

### ✅ Solution: Exercise 3

**Effective Prompt:**

```
> Generate a PCI-DSS compliance checklist for payment processing pipeline code review

<pci_requirements>
- Requirement 3.2: Do not store CVV after authorization
- Requirement 3.4: Render PAN unreadable (mask/encrypt)
- Requirement 3.4: Encrypt transmission of cardholder data
- Requirement 6.5.1: Protect against injection flaws
- Requirement 10.2: Implement audit trail for access to cardholder data
</pci_requirements>

Output format:

---
# PCI-DSS Compliance Checklist
**Pipeline:** payment_processing.py
**Review Date:** [today's date]
**Reviewer:** [name]

## Checklist

| PCI Req | Requirement Description | Status | Evidence/Notes | Severity if Failed |
|---------|------------------------|---------|----------------|---------------------|
| 3.2 | CVV not stored post-authorization | [ ] Pass / [ ] Fail | | CRITICAL |
| 3.4 | PAN masked in logs and displays | [ ] Pass / [ ] Fail | | CRITICAL |
| 3.4 | Cardholder data encrypted in transit | [ ] Pass / [ ] Fail | | CRITICAL |
| 6.5.1 | No SQL injection vulnerabilities | [ ] Pass / [ ] Fail | | HIGH |
| 10.2 | Audit trail for cardholder data access | [ ] Pass / [ ] Fail | | HIGH |

## Summary
- [ ] All CRITICAL checks passed
- [ ] All HIGH checks passed
- [ ] Code review APPROVED / REJECTED

## Reviewer Sign-Off
- Name: _______________
- Date: _______________
- Notes: _______________
---
```

**Why This Works:**
- ✅ Structured table format
- ✅ Clear pass/fail checkboxes
- ✅ Severity indicators
- ✅ Space for evidence and notes
- ✅ Formal sign-off section
- ✅ Ready for compliance audit

---

## Common Mistakes

### Mistake 1: Vague Format Request

**❌ Bad:**
```
> Give me the validation results in a nice format
```

**✅ Good:**
```
> Return validation results as JSON with these exact fields:
{
  "total": <count>,
  "valid": <count>,
  "invalid": <count>,
  "errors": [{"type": "<type>", "count": <count>}]
}
```

---

### Mistake 2: Missing Field Specifications

**❌ Incomplete:**
```
> Create a table showing pipeline performance
```

**✅ Complete:**
```
> Create a Markdown table with these columns:
- Pipeline Name
- Execution Time (minutes)
- Records Processed
- Cost ($)
- Status (✅/❌)

Include 3 pipelines: loader, validator, enricher
```

---

### Mistake 3: No Example Output

**❌ Without Example:**
```
> Format the test results nicely
```

**✅ With Example:**
```
> Format test results as:

## Test Results
- Total: X tests
- Passed: X (X%)
- Failed: X

### Failed Tests:
1. test_name - Error: description
2. test_name - Error: description
```

---

## Summary

In this subsection, you learned:

### Output Formatting Strategies
- ✅ JSON for structured, parseable results
- ✅ Markdown tables for human-readable reports
- ✅ Structured reports for compliance and audits
- ✅ Specify exact format to get consistent results

### JSON Best Practices
- ✅ Define complete schema with field names and types
- ✅ Use nested objects for hierarchical data
- ✅ Include summary and detail sections
- ✅ Specify numeric types (count vs. percentage)

### Markdown Table Best Practices
- ✅ Specify all column names upfront
- ✅ Include units in column headers
- ✅ Use status indicators (✅/⚠️/❌)
- ✅ Add summary rows for totals/averages

### Banking Data Engineering Applications
- ✅ Data quality reports (JSON)
- ✅ Performance benchmarks (tables)
- ✅ Compliance checklists (structured Markdown)
- ✅ Test results summaries (mixed formats)
- ✅ Impact analysis reports (tables + text)

---

## Key Takeaways

**Golden Rules:**

1. **Always specify format explicitly**
   - Don't assume Claude knows what format you want
   - Provide examples of desired output structure

2. **Match format to use case**
   - JSON → Automation, parsing, APIs
   - Markdown tables → Reports, documentation, comparisons
   - Structured text → Compliance, audits, sign-offs

3. **Define all fields upfront**
   - Field names, types, and meanings
   - Include nested structure if needed
   - Specify calculated fields

4. **Use consistent naming**
   - snake_case for JSON keys
   - Clear, descriptive field names
   - Units in field names (amount_usd, time_minutes)

---

## Next Steps

👉 **[Continue to 1.4.7: Precognition (Thinking Step-by-Step)](./04-07-precognition-step-by-step.md)**

**Practice before proceeding:**
1. ✅ Try formatting one of your recent analysis results as JSON
2. ✅ Create a performance comparison table for your pipelines
3. ✅ Generate a compliance checklist for one of your systems
4. ✅ Experiment with different output formats for the same data

---

**Related Sections:**
- [Phase 1.4.5: Separating Data and Instructions](./04-05-separating-data-instructions.md) - Input structure
- [Phase 1.4.2: Basic Prompt Structure](./04-02-basic-prompt-structure.md) - Task + Context + Format
- [Phase 1.4.7: Thinking Step-by-Step](./04-07-precognition-step-by-step.md) - Next topic

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 05_Formatting_Output_and_Speaking_for_Claude.ipynb
