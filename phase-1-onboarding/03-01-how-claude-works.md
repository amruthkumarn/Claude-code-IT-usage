# Phase 1.3.1: How Claude Code Works

**Learning Objectives:**
- Understand Claude Code's architecture
- Learn the request-to-response workflow
- Grasp key operational principles
- Distinguish between Interactive REPL and Plan Mode

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.1 and 1.2 completed

---

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Workflow: Request to Response](#workflow-request-to-response)
3. [Key Principles](#key-principles)
4. [Interactive REPL vs Plan Mode](#interactive-repl-vs-plan-mode)

---

## Architecture Overview

Claude Code combines a powerful AI language model (Sonnet 4.5) with direct access to your development environment:

```
┌─────────────────────────────────────────────────────────┐
│ YOU (Data Engineer)                                      │
│ "Generate a PySpark pipeline to process transactions"   │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ CLAUDE CODE CLI (Terminal Interface)                    │
│ - Parses your request                                    │
│ - Maintains conversation context                         │
│ - Manages tool approvals                                 │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ CLAUDE AI MODEL (Sonnet 4.5)                            │
│ - Understands data engineering context                   │
│ - Plans multi-step solutions                             │
│ - Generates PySpark code                                 │
│ - Requests tool usage                                    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ TOOLS (File Operations, Commands, Search)               │
│ - Read: Read PySpark files                              │
│ - Edit: Modify transformation logic                      │
│ - Write: Create new pipeline files                       │
│ - Bash: Run pytest, spark-submit                         │
│ - Grep: Search for functions, patterns                   │
│ - Glob: Find files (pipelines/*.py)                     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ YOUR CODEBASE (PySpark Pipelines)                       │
│ - pipelines/ingestion/*.py                              │
│ - pipelines/transformations/*.py                         │
│ - tests/*.py                                             │
│ - schemas/*.py                                           │
└─────────────────────────────────────────────────────────┘
```

**Model:** Sonnet 4.5 (currently available in AWS Bedrock for banking IT use)

---

## Workflow: Request to Response

**Step 1: You make a request**
```
> Generate a PySpark function to validate customer account numbers
```

**Step 2: Claude analyzes and plans**
- Understands: Need a validation function
- Context: Banking domain, account numbers
- Plan: Create function with proper validation rules

**Step 3: Claude requests tools**
```
🔧 Tool Use: Read
File: pipelines/validators/account_validator.py
Purpose: Check existing validation patterns

[Approve? yes/no/always]
```

**Step 4: Tool executes**
- Reads the file
- Returns content to Claude

**Step 5: Claude generates code**
```python
def validate_account_number(account_num: str) -> bool:
    """
    Validate bank account number.

    Rules:
    - Must be 10-12 digits
    - Must start with valid bank code (01-99)
    - Passes Luhn checksum
    """
    # Implementation...
```

**Step 6: Claude requests permission to edit**
```
🔧 Tool Use: Edit
File: pipelines/validators/account_validator.py
Action: Add validate_account_number function

[Approve? yes/no]
```

**Step 7: You approve → Code is written**

---

## Key Principles

### 1. Conversational
- Natural language requests
- Back-and-forth dialogue
- Iterative refinement

**Example:**
```
> Generate a PySpark pipeline to process payments

Claude: I'll create a payment processing pipeline.
        What data source will you use? (S3, JDBC, Kafka)

> S3

Claude: What's the schema for the payment data?

> account_id (string), amount (decimal), timestamp (timestamp)

Claude: Great! I'll generate the pipeline with those specifications...
```

### 2. Tool-Based
- Claude uses specific tools (Read, Edit, Write, Bash, Grep, Glob)
- Each tool has a clear purpose
- Tool usage is transparent

**Available Tools:**
| Tool | Purpose | Example |
|------|---------|---------|
| Read | Read files | Read payment_processor.py |
| Edit | Modify files | Update validation logic |
| Write | Create files | Create new test file |
| Bash | Run commands | Execute pytest |
| Grep | Search content | Find function definition |
| Glob | Find files | Locate all *.py files |

### 3. Approval-Based
- You control what Claude can do
- Permissions configurable (allow, requireApproval, deny)
- Default: Read-only, approve changes

**Approval Flow:**
```
Claude: I'd like to edit payment_processor.py
You: yes
Claude: [makes the edit]

Claude: I'd like to run pytest
You: no
Claude: [does not run the command]
```

### 4. Context-Aware
- Remembers conversation history
- Understands project structure
- Uses CLAUDE.md for project standards

**Example:**
```
> Read the payment processing pipeline

[Claude reads pipelines/payment_processor.py]

> Add error handling to that pipeline

[Claude knows "that pipeline" = payment_processor.py]
```

### 5. Iterative
- Complex tasks broken into steps
- Can course-correct based on feedback
- Learns from your approval/rejection patterns

**Example:**
```
> Optimize this slow aggregation

Claude: I'll analyze the code first...
[Reads code]

Claude: I found 3 optimization opportunities:
1. Add broadcast join
2. Partition by date
3. Cache intermediate DataFrame

Which would you like me to implement?

> All three

Claude: I'll start with broadcast join...
[Implements]

Claude: Done. Shall I proceed with partitioning?

> Yes
```

---

## Interactive REPL vs Plan Mode

### Interactive REPL (Default)

```bash
claude

# Characteristics:
# - Can read AND write files (with approval)
# - Can run commands (pytest, spark-submit)
# - Maintains session context
# - Interactive approvals
```

**Use cases:**
- Development work
- Code generation
- Bug fixing
- Running tests
- Creating documentation

**Example session:**
```
$ claude

> Generate a PySpark function to read from S3
[Claude generates code]

> Now add data quality checks
[Claude modifies the code]

> Run the tests
[Claude executes pytest]
```

### Plan Mode (Read-Only)

```bash
claude --permission-mode plan

# Characteristics:
# - Read-only access
# - Cannot modify files
# - Safe for exploring production code
# - No approval prompts (nothing to approve)
# - Great for understanding codebases
```

**Use cases:**
- Exploring unfamiliar codebases
- Analyzing production code
- Learning how something works
- Reviewing before changes

**Example session:**
```
$ claude --plan

> Analyze the payment processing pipeline
[Claude reads and explains the code]

> What optimizations would you suggest?
[Claude analyzes and provides recommendations]

> Explain the data flow from ingestion to output
[Claude traces the data flow]

# No files modified, just analysis
```

### When to Use Each Mode

**Use Interactive REPL when:**
- ✅ You want Claude to generate or modify code
- ✅ You need to run commands (tests, linters)
- ✅ You're actively developing

**Use Plan Mode when:**
- ✅ You're exploring production code
- ✅ You want analysis without risk of changes
- ✅ You're learning how existing code works
- ✅ You want to understand a complex codebase

### Switching Between Modes

```bash
# Start in Plan Mode
claude --plan
> Analyze the pipeline
[Exit with Ctrl+D]

# Switch to Interactive Mode
claude
> Now implement the optimizations we discussed
```

---

## Practical Examples

### Example 1: Understanding a Pipeline (Plan Mode)

```bash
claude --plan

> Explain the customer aggregation pipeline in pipelines/customer_aggregation.py
> What data sources does it use?
> What transformations are applied?
> Where is the output written?
> What are the potential performance bottlenecks?
```

**Benefit:** Understand complex pipelines safely without any risk of modification.

### Example 2: Building a New Feature (Interactive Mode)

```bash
claude

> Generate a PySpark pipeline to process daily transaction summaries
> - Read from S3 (s3://banking-data/transactions/)
> - Aggregate by account_id and date
> - Calculate total_amount, transaction_count
> - Write to Delta Lake

[Claude generates the code with approval]

> Add data quality checks for null values

[Claude adds validations]

> Generate pytest tests for this pipeline

[Claude creates test file]
```

**Benefit:** Full development workflow with code generation and testing.

### Example 3: Debugging (Interactive Mode)

```bash
claude

> The payment aggregation in pipelines/aggregation.py is producing incorrect totals.
> Debug and fix the issue.

[Claude reads the code, identifies the bug, suggests a fix, and applies it with your approval]

> Run the tests to verify the fix

[Claude executes pytest]
```

**Benefit:** Rapid debugging and testing cycle.

---

## Summary

In this subsection, you learned:

### Architecture
- ✅ Claude Code combines AI with direct codebase access
- ✅ Uses Sonnet 4.5 model (AWS Bedrock)
- ✅ Tool-based approach for file operations

### Workflow
- ✅ Request → Analysis → Tool Usage → Code Generation → Approval
- ✅ Transparent tool usage
- ✅ You control what gets executed

### Key Principles
- ✅ Conversational interface
- ✅ Tool-based architecture
- ✅ Approval-based workflow
- ✅ Context-aware processing
- ✅ Iterative refinement

### Modes
- ✅ Interactive REPL for development
- ✅ Plan Mode for safe exploration

---

## Next Steps

👉 **[Continue to 1.3.2: Security & Permissions Model](./03-02-security-permissions.md)**

**Quick Practice:**
1. Start Claude Code in Plan Mode: `claude --plan`
2. Ask it to explain an existing file in your project
3. Exit and restart in Interactive Mode: `claude`
4. Ask it to generate a simple function

---

**Related Sections:**
- [Phase 1.1: Introduction](./01-introduction-getting-started.md) - Why use Claude Code?
- [Phase 1.3.4: CLI Reference](./03-04-cli-reference.md) - All command-line options
- [Phase 1.3.2: Security & Permissions](./03-02-security-permissions.md) - Control what Claude can do

---

**Last Updated:** 2025-10-23
**Version:** 1.0
