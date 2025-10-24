# Phase 1.4.9: Avoiding Hallucinations

**Learning Objectives:**
- Recognize when Claude might hallucinate
- Ground prompts in actual codebase facts
- Verify Claude's suggestions against reality
- Apply verification strategies for banking code

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.8 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how grounding prevents hallucinations.

```bash
claude
```

**In the Claude REPL, compare:**

**❌ RISKY (May hallucinate):**
```
> What PySpark functions should I use for transaction validation?
```

**✅ GROUNDED (Fact-based):**
```
> Review the actual validation code in pipelines/validators/transaction_validator.py

Based ONLY on what you see in the file:
1. List the validation functions currently implemented
2. Identify any missing validations compared to requirements
3. Suggest improvements to EXISTING code (not new invented functions)

Do NOT suggest PySpark functions unless you see them imported in the file.
If you're unsure about something, say "I don't see this in the code."
```

**Exit Claude:**
```bash
Ctrl+D
```

**Key Insight:** Grounding in actual code prevents invented APIs and functions!

---

## Table of Contents
1. [What Are Hallucinations?](#what-are-hallucinations)
2. [Prevention Strategies](#prevention-strategies)
3. [Grounding in Codebase](#grounding-in-codebase)
4. [Banking Data Engineering Examples](#banking-data-engineering-examples)
5. [Practice Exercises](#practice-exercises)
6. [Summary](#summary)

---

## What Are Hallucinations?

### Definition

**Hallucination** = When Claude generates information that is plausible-sounding but incorrect or unverified.

### Common Hallucinations in Data Engineering

**1. Invented Functions/APIs:**
```python
# Claude might suggest:
df.auto_validate()  # ← This doesn't exist in PySpark!
df.smart_deduplicate()  # ← Not a real function
```

**2. Incorrect Configuration:**
```python
# Claude might claim:
spark.conf.set("spark.sql.adaptive.enabled", "yes")  # ← Should be True, not "yes"
```

**3. Wrong Library Usage:**
```python
# Claude might suggest pandas when you need PySpark:
df.groupby('account_id').sum()  # ← pandas syntax, not PySpark
```

**4. Fabricated Best Practices:**
```
# Claude might claim:
"According to PCI-DSS 4.0 section 12.7..."  # ← Section number doesn't exist
```

---

## Prevention Strategies

### Strategy 1: Give Claude an "Out"

**Allow uncertainty:**
```
> What validation functions are used in pipelines/validators/?

If you don't see specific functions in the code, say:
"I cannot determine this without reading the actual file."

If you're unsure, respond with:
"I'm not certain, but based on typical patterns..."
```

**Why this works:** Claude won't feel pressure to make up answers.

---

### Strategy 2: Require Evidence

**Ask for quotes:**
```
> Does our codebase validate transaction amounts?

First, find and quote the relevant code.
Then, answer based only on what you found.

Format:
Evidence: [exact code quote from file:line]
Answer: [based on evidence above]
```

**Example Response:**
```
Evidence: (from pipelines/validators/transaction_validator.py:45-48)
```python
def validate_amount(df: DataFrame) -> DataFrame:
    return df.filter(col('amount') > 0)
```

Answer: Yes, the codebase validates that amounts are positive (> 0).
However, I don't see validation for maximum amount limits.
```

---

### Strategy 3: Reference Actual Files

**Ground in reality:**
```
> Suggest improvements to our validation logic

IMPORTANT: Only reference files and functions that actually exist.

Available files (confirmed):
- pipelines/validators/transaction_validator.py
- pipelines/validators/schema_validator.py
- schemas/transaction.py

Do NOT invent new files or functions.
Base suggestions on IMPROVING existing code.
```

---

### Strategy 4: Ask for Verification Steps

**Include testing:**
```
> Add currency validation to our pipeline

After providing the code:
1. List the imports needed (verify they're standard PySpark)
2. Suggest how to TEST this change works
3. Note any ASSUMPTIONS you made

If you're making assumptions, explicitly state:
"Assumption: The currency field exists in the schema"
```

---

### Strategy 5: Limit Scope to Facts

**Constrain to observable facts:**
```
> What compliance requirements does our pipeline meet?

Answer based ONLY on:
- Code comments mentioning compliance (PCI-DSS, SOX, GDPR)
- Validation rules that map to compliance requirements
- Audit logging implementations you can see

Do NOT assume compliance is met unless you see explicit evidence.
Format: "I see evidence of [X]" or "I do NOT see evidence of [Y]"
```

---

## Grounding in Codebase

### Technique 1: File-First Prompts

**Pattern:**
```
> Read [FILE_PATH]

Based ONLY on this file:
1. [Question about what's actually there]
2. [Question comparing to requirements]
3. [Suggestions for improvement]

Do not reference code from other files unless I provide them.
```

**Example:**
```
> Read pipelines/etl/transaction_loader.py

Based ONLY on this file:
1. What data sources does it read from?
2. What transformations does it apply?
3. Are there any hardcoded values that should be configurable?

Cite line numbers for each finding.
```

---

### Technique 2: Diff-Based Suggestions

**Pattern:**
```
> Here's the current code:
[paste actual code]

Suggest improvements as diffs:
- Show BEFORE (actual current code)
- Show AFTER (your suggestion)
- Explain WHAT changed and WHY

Do not rewrite entire functions - only show the changed lines.
```

---

### Technique 3: Constraint-Based Prompts

**Pattern:**
```
> Optimize this PySpark join

Constraints:
- MUST use only standard PySpark functions (no custom UDFs)
- MUST maintain current output schema
- MUST work with PySpark 3.5+ (our version)
- CANNOT use deprecated functions (e.g., no RDD API)

If a constraint cannot be met, explain why.
```

---

## Banking Data Engineering Examples

### Example 1: Code Review Grounded in Actual Code

**Prompt:**
```
> Review pipelines/payment_processor.py for PCI-DSS compliance

Evidence-based review:

For each finding, provide:
1. **Evidence:** Quote the exact code (file:line)
2. **Issue:** What PCI-DSS requirement it violates
3. **Fix:** Specific code change (not generic advice)
4. **Verification:** How to test the fix works

PCI-DSS requirements to check:
- Requirement 3.2: No CVV storage
- Requirement 3.4: PAN masking in logs
- Requirement 10.2: Audit trail

If you DON'T find evidence of a violation, state:
"No evidence of CVV storage in this file."

If you're UNCERTAIN, state:
"Cannot determine without seeing [dependency file]."
```

**Good Response:**
```
Finding 1: PAN Exposure in Logs
================================
Evidence: (payment_processor.py:67)
```python
logger.info(f"Processing transaction {txn_id} for card {card_number}")
```

Issue: Violates PCI-DSS 3.4 - Full card number logged in plaintext

Fix:
```python
masked_card = card_number[-4:].rjust(len(card_number), '*')
logger.info(f"Processing transaction {txn_id} for card {masked_card}")
```

Verification:
1. Add test case that checks log output
2. Assert log contains "***1234" not full card number
3. Run: pytest tests/test_payment_processor.py::test_card_masking_in_logs

Finding 2: Audit Trail
======================
No evidence of audit logging in this file.

Cannot determine if audit trail exists without seeing:
- audit/audit_logger.py (if it exists)
- Database audit table schema

Recommendation: Confirm audit logging implementation location.
```

**Bad Response (Hallucination):**
```
Finding: Use the pyspark_pci_compliance library  ← INVENTED LIBRARY!
```python
from pyspark_pci_compliance import auto_mask_pan  ← DOESN'T EXIST
```
```

---

### Example 2: API Verification

**Prompt:**
```
> Suggest PySpark optimizations for this aggregation

<current_code>
```python
daily_totals = (
    transactions_df
    .groupBy('account_id', 'date')
    .agg(sum('amount').alias('total'))
)
```
</current_code>

Requirements:
- Suggest ONLY standard PySpark 3.5 functions
- For each suggestion, cite the official PySpark documentation
- If you're unsure a function exists, say so

Format:
Optimization: [description]
Code: [example using REAL PySpark functions]
Documentation: [function name from pyspark.sql.functions]
Verification: [how to confirm this works]
```

---

### Example 3: Compliance Verification

**Prompt:**
```
> Verify our transaction pipeline meets SOX compliance for audit trails

Evidence-based verification:

For each SOX requirement, provide:
✅ MET - [Quote code showing compliance]
❌ NOT MET - [Explain what's missing]
❓ UNCERTAIN - [What you'd need to see to verify]

SOX Requirements (Section 404):
1. All data changes must be logged
2. User ID must be captured for each change
3. Timestamp must be recorded for each change
4. Original and new values must be logged
5. Logs must be tamper-proof (append-only)

Base your assessment ONLY on code you can see.
Do NOT assume logging exists unless you see explicit implementation.
```

**Good Response:**
```
SOX Section 404 Compliance Assessment
======================================

1. All data changes must be logged
STATUS: ❓ UNCERTAIN

Observed: I see UPDATE statements in pipelines/transaction_processor.py:89
```python
transactions_df.write.mode('overwrite').save(output_path)
```

Missing: I do NOT see any logging calls before or after this write.

Need to see:
- audit/logger.py implementation
- Database audit_log table usage
- Trigger or hook that captures writes

2. User ID must be captured
STATUS: ❌ NOT MET

Evidence: (pipelines/transaction_processor.py:1-100)
- No imports of user context or authentication
- No references to user_id, current_user, or similar
- No environment variables for user tracking

Required: Add user context to pipeline execution
```python
audit_context = {
    'user_id': os.getenv('PIPELINE_USER_ID'),
    'execution_id': str(uuid.uuid4()),
    'timestamp': datetime.now()
}
```

3. Timestamp recorded
STATUS: ✅ MET (partially)

Evidence: (pipelines/transaction_processor.py:56)
```python
.withColumn('processed_timestamp', current_timestamp())
```

Note: This timestamps the PROCESSING, but may not be sufficient for
      SOX audit requirements which need MODIFICATION timestamps.

Recommendation: Verify with compliance team if processed_timestamp
                meets SOX requirements.
```

---

## Practice Exercises

### 📝 Exercise 1: Detect Hallucination Risk

**Scenario:** Identify which prompt is likely to cause hallucinations.

**Prompt A:**
```
> What's the best way to validate banking transactions in PySpark?
```

**Prompt B:**
```
> Review pipelines/validators/transaction_validator.py
> Based on the actual code, suggest improvements to the validation logic
```

**Question:** Which prompt has higher hallucination risk? Why?

---

### ✅ Solution: Exercise 1

**Answer:** Prompt A has MUCH higher hallucination risk.

**Why:**
- Prompt A is generic - Claude may invent "best practices" or libraries
- Prompt A doesn't ground Claude in actual code
- Prompt A might get answers based on common patterns, not your codebase

Prompt B is grounded because:
- ✅ References specific file
- ✅ Asks about ACTUAL code
- ✅ Requests improvements to EXISTING logic (not invented approaches)

**Better version of Prompt A:**
```
> I need to validate banking transactions in PySpark

Before suggesting approaches:
1. Ask me what validation rules I need
2. Ask what my current schema is
3. Then suggest code using ONLY standard PySpark functions

If you're unsure if a PySpark function exists, ask me to confirm.
```

---

### 📝 Exercise 2: Add Evidence Requirements

**Your Task:** Rewrite this prompt to require evidence and prevent hallucinations.

**Original Prompt:**
```
> Does our pipeline comply with PCI-DSS?
```

---

### ✅ Solution: Exercise 2

**Improved Prompt:**
```
> Assess PCI-DSS compliance of our payment processing pipeline

Evidence-based assessment:

For each PCI-DSS requirement below, provide:
1. **Evidence:** Quote actual code showing compliance (file:line)
2. **Assessment:** COMPLIANT / NON-COMPLIANT / CANNOT DETERMINE
3. **Reasoning:** Why you reached this conclusion

Requirements to check:
- PCI-DSS 3.2: CVV not stored after authorization
- PCI-DSS 3.4: PAN masked in logs and non-production environments
- PCI-DSS 6.5.1: No SQL injection vulnerabilities
- PCI-DSS 10.2: Audit trail for access to cardholder data

Files to review:
- pipelines/payment_processor.py
- pipelines/loaders/transaction_loader.py

IMPORTANT:
- If you don't see evidence, say "CANNOT DETERMINE - no evidence in provided files"
- Do NOT assume compliance without seeing explicit code
- If you need to see additional files to assess, list them
```

---

## Common Hallucination Patterns

### Pattern 1: Invented Libraries

**Hallucination:**
```python
from pyspark_banking_utils import validate_transaction  # ← Doesn't exist!
```

**Prevention:**
```
> Use ONLY these libraries:
- pyspark.sql.functions (standard PySpark)
- pyspark.sql.types (standard PySpark)
- decimal (Python standard library)

Do NOT import any third-party banking/validation libraries.
```

---

### Pattern 2: Fabricated Configuration

**Hallucination:**
```python
spark.conf.set("spark.pci.compliance.enabled", True)  # ← Not a real config!
```

**Prevention:**
```
> Suggest Spark configurations

For each config:
- Verify it exists in Spark 3.5 documentation
- Provide the configuration key exactly as documented
- Include default value and recommended value
```

---

### Pattern 3: Wrong API Usage

**Hallucination:**
```python
# Mixing pandas and PySpark:
df.groupby('account_id').sum()  # ← pandas syntax!
```

**Prevention:**
```
> Optimize this PySpark code

Requirements:
- Use PySpark DataFrame API (not pandas, not RDD)
- All functions must be from pyspark.sql.functions
- Use method chaining with .function() not function(df)

Example of CORRECT syntax:
df.groupBy('account_id').agg(sum('amount'))  # ← PySpark
```

---

## Summary

### Hallucination Prevention
- ✅ Give Claude permission to say "I don't know"
- ✅ Require evidence and citations
- ✅ Ground prompts in actual code
- ✅ Constrain to known libraries/APIs
- ✅ Ask for verification steps

### Evidence-Based Prompting
- ✅ Request quotes from actual code
- ✅ Ask for file:line references
- ✅ Require documentation citations
- ✅ Separate facts from assumptions

### Banking Code Safety
- ✅ Verify compliance claims with evidence
- ✅ Check API suggestions against docs
- ✅ Test all suggested code changes
- ✅ Don't trust invented libraries

---

## Key Takeaways

**Hallucination-Resistant Prompt Pattern:**
```
> [TASK]

Evidence requirements:
- Quote actual code with file:line references
- Use ONLY [list of allowed libraries]
- If uncertain, say "I cannot determine this without [X]"

Verification:
- How to test this works
- What assumptions you made

If you don't see evidence of [X], explicitly state:
"I do NOT see evidence of [X] in the provided code."
```

**Red Flags:**
- Generic "best practices" without code references
- Invented library/function names
- Configuration keys you don't recognize
- Compliance claims without specific code evidence

---

## Next Steps

👉 **[Continue to 1.4.10: Complex Prompts from Scratch](./04-10-complex-prompts.md)**

**Practice:**
1. ✅ Review a Claude suggestion - did it hallucinate?
2. ✅ Rewrite one of your prompts to require evidence
3. ✅ Test suggested code changes before applying them

---

**Related Sections:**
- [Phase 1.4.7: Thinking Step-by-Step](./04-07-precognition-step-by-step.md) - Systematic analysis
- [Phase 1.4.8: Using Examples](./04-08-using-examples.md) - Show don't tell
- [Phase 1.3.3: Memory & Context](../03-03-memory-context.md) - Grounding in project

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - 08_Avoiding_Hallucinations.ipynb
