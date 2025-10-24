# Phase 1.3.1: How Claude Code Works

**Learning Objectives:**
- Understand Claude Code's architecture
- Learn the request-to-response workflow
- Grasp key operational principles
- Distinguish between Interactive REPL and Plan Mode

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.1 and 1.2 completed

---

## ⚡ Quick Start (2 minutes)

**Goal:** Experience Claude Code in action before diving into theory.

### Try This Right Now

```bash
# 1. Start Claude Code (read-only mode is safe)
claude --permission-mode plan

# 2. Try these commands
> What version of Claude Code am I using?

> List all Python files in the current directory

> Explain what Claude Code can do

# 3. Exit
Ctrl+D
```

**What just happened?**
- You started Claude Code in "Plan Mode" (read-only, safe)
- Claude responded to natural language questions
- No files were modified—just exploration

**Next:** Let's understand the architecture that makes this possible...

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

## 🔨 Exercise 1: Your First Claude Code Command (3 minutes)

**Goal:** Understand the request → response flow by experiencing it.

### Step 1: Start Claude Code
```bash
cd ~/projects  # Or any directory with code
claude --permission-mode plan
```

### Step 2: Ask Claude to analyze a file
```
> Find all Python files in this directory and tell me what they do
```

**What you'll see:**
1. Claude uses the **Glob tool** to find *.py files
2. Claude uses the **Read tool** to read file contents
3. Claude summarizes what each file does

### Step 3: Ask a follow-up question
```
> Which file is the largest?
```

**What happened:** Claude remembered the context from Step 2 (session memory)

### ✅ Checkpoint
- [ ] You started Claude Code successfully
- [ ] You saw Claude use tools (Glob, Read)
- [ ] You asked a follow-up question and Claude remembered context

**Key Insight:** You just experienced the full architecture in action: CLI → AI Model → Tools → Your Codebase

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

## 🔨 Exercise 2: Watch the Workflow in Action (5 minutes)

**Goal:** See the 7-step workflow happen in real-time.

### Step 1: Start Claude Code in Interactive Mode
```bash
cd ~/practice-project  # Any test directory
claude  # Interactive mode (not plan mode this time)
```

### Step 2: Request code generation
```
> Create a simple Python function called validate_amount that:
> - Takes an amount (float) as input
> - Returns True if amount > 0 and amount <= 10000
> - Returns False otherwise
> - Save it to validators/amount.py
```

### Step 3: Watch the workflow unfold

**You'll see each step:**
1. **Claude analyzes** your request
2. **Claude plans** the solution
3. **Claude requests Write tool** to create the file
4. **You get an approval prompt** - Type `yes`
5. **Tool executes** - File is created
6. **Claude confirms** the file was created

### Step 4: Test the workflow with a modification
```
> Add a docstring to that function explaining the validation rules
```

**Watch for:**
- Claude uses **Read tool** (to read the file)
- Claude uses **Edit tool** (to modify it)
- You approve with `yes`

### ✅ Checkpoint
- [ ] You saw the 7-step workflow in action
- [ ] You approved a Write tool use
- [ ] You approved an Edit tool use
- [ ] You understand how tools chain together

### 💻 Terminal Session Example

Here's exactly what you'll see:

```bash
$ claude

> Create a simple Python function...

🤖 I'll create a validation function for amounts.

🔧 Tool Use: Write
File: validators/amount.py
[Shows preview of code to be written]

Approve this write? (yes/no) yes

✅ Created: validators/amount.py

> Add a docstring...

🔧 Tool Use: Read
File: validators/amount.py
[Reading the file...]

🔧 Tool Use: Edit
File: validators/amount.py
[Shows diff of changes]

Approve this edit? (yes/no) yes

✅ Updated: validators/amount.py
```

**Key Insight:** Every tool use is transparent and requires your approval (unless configured otherwise).

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

## 🔨 Exercise 3: Test the Approval Workflow (5 minutes)

**Goal:** Experience Claude's approval-based safety mechanism.

### Step 1: Start in Interactive Mode
```bash
claude
```

### Step 2: Try to create a file (will require approval)
```
> Create a file called test.txt with the content "Hello World"
```

**Expected:**
- Claude will request **Write tool** permission
- You'll see a prompt: `Approve this write? (yes/no)`

### Step 3: Test different approval responses

**Try "yes":**
```
Approve this write? (yes/no) yes
```
→ File is created

**Try "no":**
```
> Create another file called test2.txt
Approve this write? (yes/no) no
```
→ File is NOT created, Claude respects your decision

**Try "always":**
```
> Create test3.txt
Approve this write? (yes/no) always
```
→ File is created, and **Write tool is auto-approved for the rest of this session**

### Step 4: Test auto-approval
```
> Create test4.txt
```

**Expected:** No approval prompt this time (because you said "always" in Step 3)

### ✅ Checkpoint
- [ ] You approved a tool use with "yes"
- [ ] You rejected a tool use with "no"
- [ ] You auto-approved with "always"
- [ ] You understand approval lasts only for the current session

### 🎯 Challenge: What happens when you restart?

Exit Claude Code (`Ctrl+D`) and restart it. Then try:
```
> Create test5.txt
```

**Question:** Will you need to approve again?

<details>
<summary>💡 Answer</summary>

**Yes!** The "always" approval only lasts for that session. When you restart Claude Code, you're back to requiring approval for each tool use.

**Banking IT Benefit:** This prevents accidental auto-approvals from persisting across sessions, maintaining security.

</details>

**Key Insight:** You have full control over what Claude can do, with approval granularity down to individual operations.

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

## 🔨 Exercise 4: Compare Plan Mode vs Interactive Mode (10 minutes)

**Goal:** Understand when to use each mode through direct comparison.

### Part A: Plan Mode (Safe Exploration)

**Step 1: Start in Plan Mode**
```bash
cd ~/claude-code-documentation  # Or any project
claude --permission-mode plan
```

**Step 2: Try to analyze files**
```
> Read the README.md file and summarize what this project does
```

**Expected:** ✅ Works! Plan mode allows reading files.

**Step 3: Try to modify a file**
```
> Add a new section to the README.md called "Installation"
```

**Expected:** ❌ Claude will say it can't modify files in Plan Mode.

**Step 4: Exit**
```
Ctrl+D
```

### Part B: Interactive Mode (Full Control)

**Step 1: Start in Interactive Mode**
```bash
claude
```

**Step 2: Try the same modification**
```
> Add a new section to the README.md called "Installation"
```

**Expected:** ✅ Claude will request Edit tool permission. Approve with `yes`.

### Comparison Table

| Task | Plan Mode | Interactive Mode |
|------|-----------|------------------|
| Read files | ✅ Allowed | ✅ Allowed |
| Search files | ✅ Allowed | ✅ Allowed |
| Edit files | ❌ Denied | ✅ With approval |
| Create files | ❌ Denied | ✅ With approval |
| Run commands | ❌ Denied | ✅ With approval |
| Best for | Analysis, Learning | Development, Changes |

### ✅ Checkpoint
- [ ] You used Plan Mode to safely analyze code
- [ ] You saw that Plan Mode blocks modifications
- [ ] You used Interactive Mode to make changes
- [ ] You understand when to use each mode

### 🎯 Real-World Scenarios

**Use Plan Mode when:**
```bash
# Scenario 1: Exploring a new codebase at work
claude --plan
> Analyze the payment processing pipeline and explain how it works

# Scenario 2: Reviewing production code
claude --plan
> Check for potential security issues in this authentication module

# Scenario 3: Understanding legacy code
claude --plan
> Trace the data flow from input to output in this ETL process
```

**Use Interactive Mode when:**
```bash
# Scenario 1: Building a new feature
claude
> Generate a new PySpark transformation for customer data enrichment

# Scenario 2: Fixing bugs
claude
> Fix the null pointer exception in pipelines/aggregation.py line 145

# Scenario 3: Writing tests
claude
> Generate pytest tests for the validation module
```

**Key Insight:** Plan Mode is your "read-only safety net" for exploration. Interactive Mode is for active development.

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

## 🔨 Exercise 5: Complete End-to-End Workflow (15 minutes)

**Goal:** Put everything together in a realistic banking data engineering scenario.

### Scenario
You need to create a PySpark validation function for transaction amounts in a banking pipeline.

### Step 1: Explore existing code (Plan Mode)
```bash
cd ~/banking-project  # Or create a practice directory
claude --permission-mode plan

> Find all files related to validation in this project
> Show me examples of existing validation patterns
```

**Expected:** Claude will search your codebase and show you existing patterns.

### Step 2: Switch to development (Interactive Mode)
```bash
Ctrl+D  # Exit Plan Mode
claude  # Start Interactive Mode
```

### Step 3: Generate the validation function
```
> Create a PySpark validation function in validators/transaction_validator.py

Requirements:
- Function name: validate_transaction_amount
- Input: DataFrame with 'amount' column (DecimalType)
- Validation rules:
  * Amount must be positive (> 0)
  * Amount must not exceed $10,000 (daily limit)
  * Amount must have exactly 2 decimal places
- Output: Tuple of (valid_df, invalid_df)
- Include comprehensive docstring
- Add logging for validation failures
- Follow PEP 8 style
```

### Step 4: Review and approve
**Expected workflow:**
1. Claude may **Read** existing files to understand patterns
2. Claude will request **Write** tool to create the file
3. You review the generated code
4. Approve with `yes`

### Step 5: Add unit tests
```
> Generate pytest tests for this validation function

Include tests for:
- Valid transactions (amount = 100.50)
- Zero amount (should be invalid)
- Negative amount (should be invalid)
- Amount exceeding limit (amount = 15000.00)
- Edge case: exactly at limit (amount = 10000.00)

Save tests to tests/test_transaction_validator.py
```

### Step 6: Run the tests
```
> Run pytest on the test file we just created
```

**Expected:** Claude will request **Bash** tool permission to run `pytest tests/test_transaction_validator.py`

### ✅ Checkpoint
- [ ] You used Plan Mode to explore
- [ ] You switched to Interactive Mode for development
- [ ] You generated a validation function with approval
- [ ] You generated tests
- [ ] You ran tests using Claude
- [ ] All tests passed ✅

### 💻 Complete Terminal Session

Here's what the full workflow looks like:

```bash
# ===== PART 1: EXPLORATION =====
$ claude --plan
Claude Code v1.0.0 (Plan Mode)

> Find all validation files

🔧 Using Glob tool to search...
Found: validators/account_validator.py

🔧 Reading file to show examples...
[Shows existing validation patterns]

Ctrl+D


# ===== PART 2: DEVELOPMENT =====
$ claude
Claude Code v1.0.0

> Create a PySpark validation function in validators/transaction_validator.py...

🤖 I'll create a validation function following the patterns I see in your codebase.

🔧 Tool Use: Write
File: validators/transaction_validator.py
Preview:
[Shows the complete function code]

Approve this write? (yes/no) yes

✅ Created: validators/transaction_validator.py

> Generate pytest tests...

🔧 Tool Use: Write
File: tests/test_transaction_validator.py
Preview:
[Shows the test code]

Approve this write? (yes/no) yes

✅ Created: tests/test_transaction_validator.py

> Run pytest on the test file

🔧 Tool Use: Bash
Command: pytest tests/test_transaction_validator.py -v

Approve this command? (yes/no) yes

Running tests...

========== test session starts ==========
collected 5 items

tests/test_transaction_validator.py::test_valid_amount PASSED
tests/test_transaction_validator.py::test_zero_amount PASSED
tests/test_transaction_validator.py::test_negative_amount PASSED
tests/test_transaction_validator.py::test_amount_exceeds_limit PASSED
tests/test_transaction_validator.py::test_amount_at_limit PASSED

========== 5 passed in 0.23s ==========

✅ All tests passed!
```

### 🎯 Challenge: Add More Validation

Now try this on your own:
```
> Add validation to reject transactions with more than 2 decimal places
> Update the tests to cover this new rule
> Run the tests again
```

<details>
<summary>💡 Hint</summary>

The prompt structure should be:
1. Clear task ("Add validation for...")
2. Specific rule (exactly 2 decimal places)
3. Test coverage requirement
4. Run tests to verify

</details>

### Key Takeaway

**You just experienced the complete Claude Code workflow:**
1. ✅ Exploration (Plan Mode)
2. ✅ Code generation (Interactive Mode + Write)
3. ✅ Iterative development (Edit)
4. ✅ Testing (Bash tool)
5. ✅ Verification

**Banking IT Application:** This workflow ensures every generated function includes validation, tests, and verification - critical for financial data processing.

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
