# Phase 1.4.12: Tool Use (Appendix)

**Learning Objectives:**
- Leverage Claude Code tools effectively in prompts
- Direct Claude to use Read, Edit, Glob, Grep tools
- Apply tool-based workflows to code refactoring
- Master agent delegation for complex tasks

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.11 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how directing tool use improves efficiency.

```bash
claude
```

**In the Claude REPL:**

**❌ WITHOUT Tool Direction:**
```
> Fix the validation logic in the codebase
```

**✅ WITH Tool Direction:**
```
> Fix validation logic using this workflow:

1. Use Glob to find all *validator*.py files
2. Use Read to examine transaction_validator.py
3. Identify the validation issue
4. Use Edit to fix the specific function
5. Confirm the fix by re-reading the edited section
```

**Exit Claude:**
```bash
Ctrl+D
```

**Key Insight:** Directing which tools to use leads to more efficient, accurate results!

---

## Table of Contents
1. [Claude Code Tools Overview](#claude-code-tools-overview)
2. [Tool-Directed Prompts](#tool-directed-prompts)
3. [Banking Data Engineering Examples](#banking-data-engineering-examples)
4. [Practice Exercises](#practice-exercises)
5. [Summary](#summary)

---

## Claude Code Tools Overview

### Available Tools

Claude Code has these main tools:

| Tool | Purpose | Best For |
|------|---------|----------|
| **Read** | Read file contents | Reviewing code, schemas, configs |
| **Write** | Create new files | Generating new modules, tests |
| **Edit** | Modify existing files | Refactoring, bug fixes |
| **Glob** | Find files by pattern | Discovering files, finding components |
| **Grep** | Search file contents | Finding function usage, patterns |
| **Bash** | Execute commands | Running tests, git operations |
| **Task** | Delegate to agents | Complex multi-step tasks |

### When to Direct Tool Use

**Direct tool use when:**
- You want a specific workflow (Read → Analyze → Edit)
- Working with multiple files
- Need systematic code exploration
- Refactoring patterns across codebase

**Let Claude choose tools when:**
- Simple single-file tasks
- Claude knows the best approach
- Exploratory tasks

---

## Tool-Directed Prompts

### Pattern 1: Read → Analyze → Edit

**Use Case:** Fix a bug in existing code

**Prompt:**
```
> Fix the null pointer exception in transaction_validator.py

Follow this workflow:
1. Read pipelines/validators/transaction_validator.py
2. Identify where null values could cause issues
3. Edit the file to add null checks before validation
4. Confirm fix by re-reading the modified section

Start with step 1 now.
```

**Why This Works:**
- Explicit workflow prevents Claude from guessing
- Each step is verifiable
- Can catch issues at each stage

---

### Pattern 2: Glob → Grep → Read

**Use Case:** Find and understand code patterns

**Prompt:**
```
> Find all validation functions in our codebase

Workflow:
1. Use Glob to find all files matching pattern: **/*validator*.py
2. Use Grep to search for 'def validate_' in those files
3. Read the top 3 validator files to understand the pattern
4. Summarize the validation pattern we're using

Provide results from each step before proceeding to next.
```

---

### Pattern 3: Read Multiple → Compare → Edit

**Use Case:** Maintain consistency across files

**Prompt:**
```
> Update all validator functions to use the same error handling pattern

Workflow:
1. Read pipelines/validators/transaction_validator.py (our reference)
2. Read pipelines/validators/account_validator.py
3. Read pipelines/validators/currency_validator.py
4. Compare error handling approaches
5. Edit account_validator.py and currency_validator.py to match transaction_validator.py pattern

Show diffs for each edit.
```

---

### Pattern 4: Task (Agent Delegation)

**Use Case:** Complex multi-file refactoring

**Prompt:**
```
> Refactor all validation functions to use a common base class

This is complex, so delegate to a general-purpose agent:

Task for agent:
1. Find all *validator*.py files
2. Analyze common patterns across validators
3. Design a BaseValidator class
4. Create the base class in pipelines/validators/base.py
5. Refactor each validator to inherit from BaseValidator
6. Run tests to confirm all validators still work

Agent should provide:
- Base class design
- Refactored code for each validator
- Test results
```

---

## Banking Data Engineering Examples

### Example 1: Schema Migration Across Multiple Files

**Prompt:**
```
> Add merchant_category field to all files using transaction schema

Tool-directed workflow:

Step 1: Find all files referencing transaction schema
```
Use Grep to search for 'transaction_schema' or 'StructType.*txn_id'
Output: List of files
```

Step 2: Read the current schema definition
```
Read schemas/transaction.py
Output: Current StructType definition
```

Step 3: For each file found in step 1:
```
Read [file]
Identify how schema is used
Plan how to add merchant_category field
```

Step 4: Update schema definition
```
Edit schemas/transaction.py
Add: StructField("merchant_category", StringType(), nullable=True)
```

Step 5: Update all dependent files
```
For each file from step 1:
  Edit [file] to handle new merchant_category field
  Show diff
```

Step 6: Verification
```
Use Grep to confirm merchant_category is now in all necessary places
```

Execute this workflow step-by-step. Show results at each step before proceeding.
```

---

### Example 2: Code Review with Tool Direction

**Prompt:**
```
> Review all transaction processing files for PCI-DSS compliance

Systematic review workflow:

Step 1: Discovery
```
Use Glob to find: pipelines/**/transaction*.py
List all files found
```

Step 2: Compliance Check (for each file)
```
Read [file]

Check for:
- CVV storage (should NOT exist)
- Card number logging (should be masked)
- Sensitive data in exception messages
- Proper encryption usage

Output: Compliance report per file
```

Step 3: Evidence Collection
```
For each violation found:
- Use Grep to find exact lines with issues
- Quote the problematic code
- Suggest fix
```

Step 4: Remediation
```
For critical violations only:
- Use Edit to apply fix
- Show before/after diff
```

Execute steps 1-4. Provide comprehensive compliance report at the end.
```

---

### Example 3: Refactoring with Pattern Consistency

**Prompt:**
```
> Refactor all ETL pipelines to use consistent error handling

Multi-file refactoring workflow:

Step 1: Find all ETL pipeline files
```
Glob pattern: pipelines/etl/*.py
List found files
```

Step 2: Identify reference implementation
```
Read pipelines/etl/transaction_processor.py
Find the error handling pattern (try/except blocks, logging, etc.)
Document the pattern
```

Step 3: Analyze other pipelines
```
For each file from step 1 (except reference):
  Read [file]
  Identify current error handling approach
  Determine gaps compared to reference pattern
```

Step 4: Refactor each pipeline
```
For each file needing updates:
  Edit [file]
  Apply reference error handling pattern
  Preserve business logic
  Show diff highlighting changes
```

Step 5: Verification
```
Grep all files for 'except' to verify pattern is consistent
Report any remaining inconsistencies
```

Show progress after each file is refactored.
```

---

### Example 4: Agent Delegation for Complex Analysis

**Prompt:**
```
> Analyze data flow and dependencies across the entire transaction processing system

This requires deep codebase analysis - delegate to general-purpose agent:

<agent_task>
Analyze transaction data flow from ingestion to reporting

Workflow for agent:
1. Use Glob to find all pipeline files (pipelines/**/*.py)
2. Use Grep to find DataFrame reads (spark.read, read_stream)
3. Use Grep to find DataFrame writes (write, writeStream)
4. Read key pipeline files to understand transformations
5. Build data lineage map
6. Identify dependencies between pipelines
7. Find potential circular dependencies
8. Identify performance bottlenecks in flow

Deliverables:
1. Data flow diagram (ASCII art or Mermaid)
2. Dependency graph
3. List of tables with source → transformations → destination
4. Recommendations for optimization
5. Risk areas (bottlenecks, single points of failure)
</agent_task>

Let the agent work autonomously. Review its findings when complete.
```

---

## Practice Exercises

### 📝 Exercise: Build a Tool-Directed Workflow

**Scenario:** Update all validation error messages to include field names.

**Your Task:** Write a prompt with explicit tool directions.

**Requirements:**
- Find all validator files
- Identify error message patterns
- Update messages to include field name
- Verify consistency

---

### ✅ Solution

**Effective Prompt:**
```
> Update validation error messages to include field names

Tool-directed workflow:

Step 1: Discovery
```
Glob: pipelines/validators/*.py
List all validator files found
```

Step 2: Pattern Analysis
```
Read pipelines/validators/transaction_validator.py (reference)
Identify current error message format
Note: Should be "Invalid [field_name]: [reason]"
```

Step 3: Find all error messages
```
Grep pattern: 'raise.*ValidationError|validation_error.*='
In files: All validators from step 1
List occurrences with file:line
```

Step 4: Update each validator
```
For each file:
  Read [validator_file]
  Edit to update error messages:
    Before: "Amount must be positive"
    After: "Invalid amount: must be positive"
  Show diff
```

Step 5: Verification
```
Grep for old pattern: "must be" (without field name)
Confirm no matches remain
```

Execute workflow. Report progress after each step.
```

---

## Common Mistakes

### Mistake 1: Not Specifying Tool

**❌ Vague:**
```
> Find all validators
```

**✅ Explicit:**
```
> Use Glob to find all files matching: **/*validator*.py
```

---

### Mistake 2: Wrong Tool for Job

**❌ Inefficient:**
```
> Use Read to check every file for validation functions
```

**✅ Efficient:**
```
> Use Grep to search for 'def validate_' across all Python files
> Then Read only the files with matches
```

---

### Mistake 3: No Verification Step

**❌ No Verification:**
```
> Edit all files to fix issue
```

**✅ With Verification:**
```
> Edit all files to fix issue
> Then Grep to confirm fix is applied everywhere
> Then Read one file to verify correctness
```

---

## Summary

### Tool-Directed Prompts
- ✅ Specify which tools to use
- ✅ Define workflow steps explicitly
- ✅ Include verification steps
- ✅ Show intermediate results

### Common Workflows
- ✅ Read → Analyze → Edit (single file fix)
- ✅ Glob → Grep → Read (pattern discovery)
- ✅ Read Multiple → Compare → Edit (consistency)
- ✅ Task (complex multi-step with agent)

### Best Practices
- ✅ Use Glob for file discovery
- ✅ Use Grep for content search
- ✅ Use Read before Edit
- ✅ Verify with Re-read or Grep

---

## Key Takeaways

**Tool Selection Guide:**
- **Finding files?** → Glob
- **Searching content?** → Grep
- **Understanding code?** → Read
- **Fixing code?** → Read then Edit
- **New code?** → Write
- **Complex task?** → Task (agent)

**Workflow Pattern:**
```
1. Discover (Glob/Grep)
2. Analyze (Read)
3. Plan (human decision point)
4. Execute (Edit/Write)
5. Verify (Grep/Read)
```

---

## Next Steps

👉 **[Continue to 1.4.13: Search & Retrieval](./04-13-search-retrieval.md)**

**Practice:**
1. ✅ Try a tool-directed refactoring task
2. ✅ Compare speed of directed vs. undirected
3. ✅ Document your team's common workflows

---

**Related Sections:**
- [Phase 1.3.4: CLI Reference](../03-04-cli-reference.md) - Tool details
- [Phase 1.4.11: Chaining Prompts](./04-11-chaining-prompts.md) - Sequential workflows

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - Appendix 10.2: Tool Use
