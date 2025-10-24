# Phase 1.4.11: Chaining Prompts (Appendix)

**Learning Objectives:**
- Chain multiple prompts for complex workflows
- Build iterative development patterns
- Apply prompt chaining to feature development
- Master sequential refinement techniques

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.10 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how chaining prompts enables complex workflows.

**Sequential Workflow Example:**

```bash
claude
```

**Prompt 1 - Design:**
```
> Design a transaction deduplication function

Return: Function signature and high-level approach only
```

**Prompt 2 - Implement (after reviewing design):**
```
> Implement the deduplication function using the design we just discussed

Use PySpark window functions
Return: Complete working code
```

**Prompt 3 - Test:**
```
> Generate pytest tests for the deduplication function we just implemented

Include: happy path, duplicates, edge cases
```

**Prompt 4 - Document:**
```
> Create documentation for the deduplication feature

Include: usage examples, performance notes, integration guide
```

**Exit Claude:**
```bash
Ctrl+D
```

**Key Insight:** Breaking complex tasks into sequential prompts allows review and refinement at each step!

---

## Table of Contents
1. [What is Prompt Chaining?](#what-is-prompt-chaining)
2. [Common Chaining Patterns](#common-chaining-patterns)
3. [Banking Data Engineering Workflows](#banking-data-engineering-workflows)
4. [Practice Exercises](#practice-exercises)
5. [Summary](#summary)

---

## What is Prompt Chaining?

### Definition

**Prompt chaining** = Breaking a complex task into a sequence of smaller prompts, where each builds on previous results.

**Why chain prompts?**
- ✅ Review intermediate results before proceeding
- ✅ Adjust course if early steps reveal issues
- ✅ Keep each prompt focused and clear
- ✅ Easier to debug if something goes wrong
- ✅ Natural workflow matches development process

### Chaining vs. Single Complex Prompt

**Single Complex Prompt (All-in-one):**
```
> Design, implement, test, and document a transaction validator
```

**Problems:**
- If design is wrong, whole implementation is wrong
- Can't review before implementation
- Hard to iterate

**Chained Prompts (Step-by-step):**
```
Prompt 1: > Design transaction validator (review design)
Prompt 2: > Implement based on approved design
Prompt 3: > Generate tests
Prompt 4: > Add documentation
```

**Benefits:**
- Review and approve design before coding
- Adjust implementation based on early results
- Each step is manageable
- Easy to go back and refine

---

## Common Chaining Patterns

### Pattern 1: Design → Implement → Test → Document

**The Classic Development Workflow:**

**Step 1: Design**
```
> Design a currency conversion function for transaction enrichment

Requirements:
- Support USD, EUR, GBP conversion
- Use exchange rate table
- Handle missing exchange rates gracefully

Return only:
- Function signature
- High-level algorithm
- Data structures needed
```

**Step 2: Implement (after approving design)**
```
> Implement the currency conversion function using this design:
[paste approved design]

Requirements:
- PySpark DataFrame operations
- Type hints
- Error handling
```

**Step 3: Test**
```
> Generate pytest tests for the currency_converter function

Test cases:
- Valid conversion (USD → EUR)
- Missing exchange rate (fallback)
- Invalid currency code
- Large batch conversion
```

**Step 4: Document**
```
> Create module documentation for currency_converter.py

Include:
- Function documentation (docstrings)
- Usage examples
- Performance notes
- Integration guide
```

---

### Pattern 2: Analyze → Fix → Verify

**The Debugging Workflow:**

**Step 1: Analyze**
```
> Analyze why the aggregation pipeline is running slow

<code>
[paste slow code]
</code>

<symptoms>
- Takes 45 minutes for 10M records
- Target: < 15 minutes
</symptoms>

Return:
- Top 3 performance bottlenecks
- Evidence from code
```

**Step 2: Fix (after understanding bottlenecks)**
```
> Fix the performance bottlenecks we identified:
1. Unnecessary shuffle in join
2. Missing partition pruning
3. Too many output partitions

Apply fixes and explain changes
```

**Step 3: Verify**
```
> Generate performance test to verify optimization

Test should:
- Measure execution time before/after
- Validate output matches original
- Check resource utilization
```

---

### Pattern 3: Research → Design → Implement

**The Discovery Workflow:**

**Step 1: Research**
```
> Search the codebase for existing validation patterns

Find:
- What validation functions already exist?
- What libraries/utilities are being used?
- What patterns should we follow?
```

**Step 2: Design (using research findings)**
```
> Design a new merchant_id validation function

Follow the patterns we found in:
- pipelines/validators/account_validator.py
- pipelines/validators/currency_validator.py

New requirements:
- Pattern: MERCH[A-Z0-9]{8}
- Not null validation
- Integration with existing validator framework
```

**Step 3: Implement**
```
> Implement merchant_id validator matching the approved design

Ensure it integrates with the existing validator pipeline
```

---

## Banking Data Engineering Workflows

### Workflow 1: Feature Development End-to-End

**Scenario:** Add merchant category enrichment to transaction pipeline

**Prompt Chain:**

**1. Requirements Analysis**
```
> Analyze requirements for adding merchant_category to transactions

Current state:
- transactions table has merchant_id
- merchant_dim table has merchant_id, merchant_category

Requirements:
- Enrich transactions with merchant_category via join
- Handle missing merchants (leave category as NULL)
- Optimize for performance (merchant_dim is 100MB)

Return:
- Join strategy recommendation
- Performance impact estimate
- Risks and mitigation
```

**2. Schema Design**
```
> Design schema change for merchant_category field

Based on analysis:
[paste analysis from step 1]

Provide:
- Updated StructType for transactions table
- Migration script approach (Delta Lake)
- Backward compatibility plan
```

**3. Implementation**
```
> Implement the enrichment logic

Using the schema design:
[paste schema from step 2]

Requirements:
- Use broadcast join (merchant_dim < 100MB)
- Add to existing transaction_enricher.py
- Preserve existing enrichment logic
```

**4. Testing**
```
> Generate integration tests for merchant enrichment

Test scenarios:
- Transaction with valid merchant_id
- Transaction with missing merchant_id (NULL category)
- Batch of 1000 transactions (performance test)
- Merchant_dim table missing (error handling)
```

**5. Documentation & Deployment**
```
> Create deployment plan and documentation

Include:
- Step-by-step deployment procedure
- Rollback plan
- Monitoring queries to verify enrichment is working
- FAQ for team members
```

---

### Workflow 2: Performance Investigation & Optimization

**Scenario:** Transaction aggregation is slow

**Prompt Chain:**

**1. Profiling**
```
> Profile the transaction aggregation pipeline

<code>
[paste pipeline code]
</code>

Analyze:
- Which stage takes most time?
- What Spark operations are used?
- Any shuffle operations?
- Partition count at each stage?

Return execution plan analysis
```

**2. Root Cause**
```
> Based on profiling results, identify root cause of slowness

Profiling showed:
[paste profiling results]

Determine:
- Is this a shuffle problem?
- Data skew issue?
- Memory problem?
- Wrong algorithm choice?

Provide specific evidence
```

**3. Solution Design**
```
> Design optimization strategy for the identified root cause

Root cause:
[paste root cause from step 2]

Propose:
- Specific code changes
- Configuration tuning
- Alternative algorithms if needed

Estimate performance improvement
```

**4. Implementation**
```
> Implement the optimization

Apply the design:
[paste design from step 3]

Provide:
- Updated code with changes highlighted
- Configuration changes
- Before/after comparison
```

**5. Benchmarking**
```
> Create benchmark to measure optimization impact

Benchmark should:
- Run both old and new code
- Measure execution time
- Compare output correctness
- Track memory usage
```

---

### Workflow 3: Compliance Audit & Remediation

**Scenario:** Prepare for PCI-DSS audit

**Prompt Chain:**

**1. Audit Scope**
```
> Define PCI-DSS audit scope for payment processing pipeline

Identify:
- Which pipelines handle cardholder data?
- What PCI-DSS requirements apply?
- What evidence is needed?

List all files/components in scope
```

**2. Compliance Check**
```
> Audit pipelines for PCI-DSS compliance

Files in scope:
[paste scope from step 1]

Check each requirement:
- 3.2: CVV not stored post-authorization
- 3.4: PAN masked in logs
- 6.5.1: No SQL injection
- 10.2: Audit trail present

Provide evidence or gaps for each
```

**3. Remediation Plan**
```
> Create remediation plan for compliance gaps

Gaps found:
[paste gaps from step 2]

For each gap:
- Severity (Critical/High/Medium/Low)
- Remediation action
- Estimated effort
- Implementation approach
```

**4. Implementation**
```
> Implement highest priority remediation

Gap: [paste #1 gap]

Provide:
- Code fix
- Testing approach
- Verification method
```

**5. Evidence Collection**
```
> Generate compliance evidence report

For each requirement:
- Requirement ID
- Evidence (code quotes, test results)
- Compliance status
- Last verified date

Format as audit-ready documentation
```

---

## Practice Exercises

### 📝 Exercise: Design a Prompt Chain

**Scenario:** You need to add real-time fraud alerting to your transaction pipeline.

**Your Task:** Design a 5-step prompt chain for this feature.

Steps to include:
1. Research existing alerting patterns
2. Design fraud detection rules
3. Implement detection logic
4. Add alerting integration (Kafka/SNS/etc.)
5. Create monitoring dashboard

---

## Summary

### Prompt Chaining Benefits
- ✅ Review and approve each step before proceeding
- ✅ Easier to debug - know exactly which step failed
- ✅ Natural workflow - matches development process
- ✅ Flexibility - adjust based on intermediate results

### Common Patterns
- ✅ Design → Implement → Test → Document
- ✅ Analyze → Fix → Verify
- ✅ Research → Design → Implement
- ✅ Audit → Remediate → Verify

### Banking Applications
- ✅ Feature development (end-to-end)
- ✅ Performance investigations
- ✅ Compliance audits
- ✅ Pipeline migrations
- ✅ Incident response

---

## Key Takeaways

**When to Chain Prompts:**
- Complex tasks with multiple phases
- When you want to review intermediate results
- When later steps depend on earlier decisions
- When you might need to backtrack and revise

**When NOT to Chain:**
- Simple, single-step tasks
- When all context is known upfront
- Time-sensitive quick fixes

**Best Practice:**
- Keep each prompt focused on one objective
- Reference previous results explicitly
- Review output before proceeding to next step

---

## Next Steps

👉 **[Continue to 1.4.12: Tool Use](./04-12-tool-use.md)**

**Practice:**
1. ✅ Take a recent complex task and break it into a prompt chain
2. ✅ Try the chain and note where you adjusted course
3. ✅ Document successful chains as team workflow templates

---

**Related Sections:**
- [Phase 1.4.10: Complex Prompts](./04-10-complex-prompts.md) - Alternative approach
- [Phase 1.4.7: Step-by-Step Reasoning](./04-07-precognition-step-by-step.md) - Within-prompt steps

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - Appendix 10.1: Chaining Prompts
