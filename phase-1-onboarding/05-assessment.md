# Phase 1.5: Assessment - Claude Code Mastery

**Overview:** Test your knowledge and skills from Phase 1 (Onboarding) before proceeding to Phase 2 (Build).

**Time Commitment:** 2-3 hours

**Prerequisites:** Phase 1.1-1.4 completed

---

## ⚡ Quick Self-Check (5 minutes)

**Goal:** Test if you're ready for Phase 2!

### Quick Check (Answer Yes/No):

1. ✅ Can you start Claude Code? (`claude`)
2. ✅ Can you create a CLAUDE.md file?
3. ✅ Can you write a custom slash command?
4. ✅ Can you set up permission restrictions?
5. ✅ Can you create a hook for audit logging?
6. ✅ Can you write an effective prompt with Task+Context+Format?

**All Yes?** You're ready for Phase 2: Build!
**Some No?** Review those sections and try the exercises.

---

## 🔨 Warm-Up Exercise: Complete Claude Code Setup (15 minutes)

**Goal:** Demonstrate you can set up a complete Claude Code project from scratch.

**This is a warm-up before the formal assessment. Use it to verify you're ready!**

### Challenge: Banking IT Project Setup

```bash
# Create assessment project
mkdir -p ~/claude-assessment && cd ~/claude-assessment

# TODO: Set up EVERYTHING you learned in Phase 1:
# 1. Create .claude directory structure
# 2. Configure settings.json with banking IT permissions
# 3. Create CLAUDE.md with project standards
# 4. Add a security hook (secrets detection)
# 5. Create a custom slash command (/validate)
# 6. Test the complete setup with Claude Code

# Start when ready!
```

**Success Criteria:**
- ✅ Project has proper directory structure
- ✅ Settings deny Bash, require approval for Edit/Write
- ✅ CLAUDE.md defines banking IT standards
- ✅ Hook prevents secrets from being committed
- ✅ Custom slash command works
- ✅ Claude Code starts without errors

**Time Limit:** 15 minutes

<details>
<summary>💡 Solution (expand after attempting)</summary>

```bash
# Complete setup
cd ~/claude-assessment

# 1. Directory structure
mkdir -p .claude/{commands,hooks}
mkdir -p pipelines tests config

# 2. Settings
cat > .claude/settings.json <<'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  },
  "defaultModel": "sonnet"
}
EOF

# 3. Project memory
cat > .claude/CLAUDE.md <<'EOF'
# Assessment Project

You are a banking data engineer. Follow PCI-DSS and SOX compliance.
- Never store CVV codes
- Mask PII in logs
- Use type hints and docstrings
EOF

# 4. Security hook
cat > .claude/hooks/detect-secrets.sh <<'EOF'
#!/bin/bash
if grep -rE "password\s*=" . 2>/dev/null; then
    echo "Secret detected!"
    exit 1
fi
exit 0
EOF
chmod +x .claude/hooks/detect-secrets.sh

# 5. Slash command
cat > .claude/commands/validate.md <<'EOF'
Generate a PySpark validation function with:
- Type hints
- Docstring
- Error handling
- pytest tests
EOF

# 6. Test
claude
> /validate
Ctrl+D
```
</details>

**If you completed this in 15 minutes:** You're ready for the assessment!
**If you needed help:** Review Phase 1.3 before continuing.

---

## Table of Contents
1. [Assessment Overview](#assessment-overview)
2. [Knowledge Check (Multiple Choice)](#knowledge-check-multiple-choice)
3. [Practical Exercises](#practical-exercises)
4. [Hands-On Project](#hands-on-project)
5. [Grading Rubric](#grading-rubric)
6. [Answer Key](#answer-key)

---

## Assessment Overview

### What This Assessment Covers

This assessment evaluates your mastery of:

**Phase 1.1-1.2: Fundamentals**
- Claude Code basics and installation
- CLI commands and modes
- Permission system

**Phase 1.3: Core Concepts (12 subsections)**
- Architecture and workflow
- Security and permissions
- Memory and context (CLAUDE.md)
- CLI reference
- Project configuration
- Slash commands
- Agents and sub-agents
- Hooks and automation
- MCP integration
- Git integration (manual policy)
- Standards and best practices
- Templates library

**Phase 1.4: Prompt Engineering (2 subsections completed)**
- Basic prompt structure
- Task + Context + Format formula

### Assessment Structure

| Section | Type | Questions/Tasks | Time | Points |
|---------|------|-----------------|------|--------|
| **Part A** | Multiple Choice | 25 questions | 30 min | 25 pts |
| **Part B** | Practical Exercises | 5 exercises | 60 min | 50 pts |
| **Part C** | Hands-On Project | 1 project | 60 min | 25 pts |
| **Total** | | 31 items | 2.5 hrs | 100 pts |

### Passing Criteria

- **Pass**: 70+ points (70%)
- **Proficient**: 85+ points (85%)
- **Expert**: 95+ points (95%)

**If you score below 70%:** Review the relevant sections before proceeding to Phase 2.

---

## Part A: Knowledge Check (Multiple Choice)

**Instructions:** Select the best answer for each question. Each question is worth 1 point.

### Section 1: Fundamentals (Questions 1-5)

**Question 1:** What is the primary purpose of Claude Code?
- A. To replace all human developers
- B. To assist developers with AI-powered coding in the terminal
- C. To automatically commit code to git
- D. To deploy applications to production

**Question 2:** Which command starts Claude Code in read-only mode?
- A. `claude --read-only`
- B. `claude --plan`
- C. `claude --permission-mode plan`
- D. Both B and C

**Question 3:** What is the default model used in Claude Code for banking IT?
- A. GPT-4
- B. Opus
- C. Sonnet
- D. Haiku

**Question 4:** Which tool does Claude Code use to search for file patterns?
- A. `grep`
- B. `find`
- C. `Glob`
- D. `locate`

**Question 5:** What is the keyboard shortcut to exit Claude Code?
- A. `Ctrl+C`
- B. `Ctrl+D`
- C. `Ctrl+Z`
- D. `exit`

### Section 2: Security & Permissions (Questions 6-10)

**Question 6:** In banking IT, which permission mode is recommended for production environments?
- A. `allow: ["*"]`
- B. `requireApproval: ["Edit", "Write"]`
- C. `deny: ["Bash"]`
- D. Read-only with `allow: ["Read", "Grep", "Glob"]`

**Question 7:** What file stores personal Claude Code settings that should NOT be committed to git?
- A. `.claude/settings.json`
- B. `.claude/settings.local.json`
- C. `.claude/config.json`
- D. `~/.claude/settings.json`

**Question 8:** According to banking IT policy, git operations must be:
- A. Performed manually by developers
- B. Automated by Claude Code
- C. Executed via hooks
- D. Disabled entirely

**Question 9:** Which compliance standard requires that CVV codes NEVER be stored?
- A. SOX
- B. GDPR
- C. PCI-DSS
- D. HIPAA

**Question 10:** What is the principle of least privilege in Claude Code?
- A. Give Claude maximum permissions for productivity
- B. Grant only necessary permissions for the task
- C. Deny all permissions by default
- D. Require approval for all operations

### Section 3: Memory & Context (Questions 11-15)

**Question 11:** What is the primary purpose of `CLAUDE.md`?
- A. To store Claude's conversation history
- B. To provide persistent project context and standards
- C. To configure permissions
- D. To define custom slash commands

**Question 12:** Where should `CLAUDE.md` be placed for project-wide standards?
- A. `~/.claude/CLAUDE.md` (user-level)
- B. `.claude/CLAUDE.md` or root directory (project-level)
- C. `/etc/claude/CLAUDE.md` (system-level)
- D. It doesn't matter

**Question 13:** What type of information should be in `CLAUDE.md` for a banking data engineering project?
- A. API keys and passwords
- B. Coding standards, PySpark patterns, compliance requirements
- C. User's personal preferences
- D. Complete codebase copy

**Question 14:** How does Claude prioritize multiple memory sources?
- A. System > Project > User
- B. User > Project > System
- C. Project > User > System
- D. All have equal priority

**Question 15:** What should you include in `CLAUDE.md` to help Claude generate PCI-DSS compliant code?
- A. Credit card test data
- B. PCI-DSS requirements (no CVV storage, tokenization, masking)
- C. Production database credentials
- D. Customer PII examples

### Section 4: Configuration & Commands (Questions 16-20)

**Question 16:** Slash commands are stored in which directory?
- A. `.claude/commands/`
- B. `.claude/slash/`
- C. `~/.claude/bin/`
- D. `/usr/local/claude/commands/`

**Question 17:** In a slash command markdown file, what does `$1` represent?
- A. The first line of output
- B. The first argument passed to the command
- C. Dollar sign followed by number 1
- D. A variable replacement

**Question 18:** What is the correct format for defining a slash command?
- A. JSON file with command definition
- B. Markdown file with frontmatter and prompt
- C. Python script in commands directory
- D. Shell script with .sh extension

**Question 19:** How do you invoke a custom slash command named "security-review"?
- A. `claude security-review`
- B. `> @security-review`
- C. `> /security-review`
- D. `> security-review`

**Question 20:** What field in settings.json specifies which tools require user approval?
- A. `"approval": ["Edit", "Write"]`
- B. `"requireApproval": ["Edit", "Write"]`
- C. `"require_approval": ["Edit", "Write"]`
- D. `"needsApproval": ["Edit", "Write"]`

### Section 5: Advanced Features (Questions 21-25)

**Question 21:** What is the primary purpose of agents (sub-agents) in Claude Code?
- A. To run background tasks
- B. To provide specialized, focused AI assistants for specific tasks
- C. To deploy code automatically
- D. To monitor system performance

**Question 22:** Hooks in Claude Code run:
- A. Only after tool execution
- B. Only before tool execution
- C. At specific points in the workflow (PreToolUse, PostToolUse, etc.)
- D. Continuously in the background

**Question 23:** What does MCP stand for in Claude Code?
- A. Model Context Protocol
- B. Multi-Cloud Platform
- C. Master Control Program
- D. Message Communication Protocol

**Question 24:** In the prompt engineering formula, what are the three essential components?
- A. Code + Tests + Documentation
- B. Task + Context + Format
- C. Input + Process + Output
- D. Read + Write + Execute

**Question 25:** When should you use read-only mode (`--permission-mode plan`)?
- A. When writing production code
- B. When exploring, analyzing, or debugging without making changes
- C. When committing to git
- D. Never - it's deprecated

---

## Part B: Practical Exercises (50 points)

**Instructions:** Complete each exercise by providing the requested configuration, code, or prompt. Write your answers in a text file.

### Exercise 1: Project Configuration (10 points)

**Scenario:** You're setting up Claude Code for a new banking data pipeline project.

**Task:** Create a `.claude/settings.json` file that:
1. Allows Read, Grep, Glob operations without approval
2. Requires approval for Edit and Write operations
3. Denies Bash operations (banking IT policy)
4. Sets default model to Sonnet
5. Defines environment variables: `SPARK_ENV=development`, `PYSPARK_PYTHON=python3`
6. Includes a PreToolUse hook that runs `./.claude/hooks/audit-log.py` for all tools

**Deliverable:** Complete `settings.json` configuration

---

### Exercise 2: CLAUDE.md Creation (10 points)

**Scenario:** Your team needs a standardized `CLAUDE.md` for a payment processing pipeline project.

**Task:** Create a `CLAUDE.md` file that includes:
1. Project overview (payment processing for banking)
2. Technology stack (Python 3.10+, PySpark 3.5+, Delta Lake)
3. Coding standards (PEP 8, type hints, docstrings)
4. PySpark best practices (use DataFrame API, avoid collect(), broadcast small tables)
5. Security requirements (mask PII in logs, use Decimal for money, parameterized queries)
6. PCI-DSS compliance notes (no CVV storage, tokenize card data)
7. Example of how to mask account numbers in logs

**Deliverable:** Complete `CLAUDE.md` file

---

### Exercise 3: Custom Slash Command (10 points)

**Scenario:** Your team frequently needs to generate pytest test cases for PySpark data validation functions.

**Task:** Create a slash command file `.claude/commands/generate-tests.md` that:
1. Has description: "Generate pytest tests for PySpark validation functions"
2. Takes one argument: the file path to test
3. Prompts Claude to generate comprehensive tests including:
   - Happy path with sample DataFrames
   - Edge cases (nulls, empty datasets, schema mismatches)
   - Error conditions
   - Banking-specific scenarios (negative amounts, duplicate transactions)
4. Specifies using `pyspark.testing.utils.assertDataFrameEqual`
5. Requests 80% code coverage

**Deliverable:** Complete slash command markdown file

---

### Exercise 4: Agent Definition (10 points)

**Scenario:** You want to create a specialized agent for PCI-DSS compliance auditing of data pipelines.

**Task:** Define an agent in `settings.json` named "compliance-checker" that:
1. Has description: "PCI-DSS compliance auditor for data pipelines"
2. Has a detailed prompt instructing it to check for:
   - CVV storage violations
   - Unencrypted card data
   - Card numbers in logs or DataFrames
   - Missing tokenization
   - Proper data masking
3. Is restricted to Read, Grep, Glob tools only (read-only)
4. Instructs the agent to provide file:line references and severity ratings

**Deliverable:** Agent configuration JSON snippet

---

### Exercise 5: Effective Prompt (10 points)

**Scenario:** You need Claude to generate a PySpark function that validates banking transactions.

**Task:** Write an effective prompt using the Task + Context + Format formula that:

**TASK:** Create a validation function for transaction DataFrames

**CONTEXT:**
- Input schema: txn_id (String), account_id (String), amount (Decimal), currency (String), timestamp (Timestamp)
- Validation rules:
  - amount must be positive
  - amount must not exceed $10,000
  - currency must be USD, EUR, or GBP
  - txn_id must be unique
- Technology: PySpark 3.5+

**FORMAT:**
- Function signature: `def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]`
- Return tuple of (valid_df, invalid_df)
- Include type hints and docstring
- Add logging for validation failures
- Use DataFrame operations (no collect())

**Deliverable:** Complete prompt text

---

## Part C: Hands-On Project (25 points)

**Instructions:** Complete this mini-project using Claude Code. Document your process and submit the generated files.

### Project: Banking Transaction Validator

**Objective:** Create a complete transaction validation pipeline with Claude Code's assistance.

**Requirements:**

1. **Schema Definition** (5 points)
   - Create `schemas/transaction.py`
   - Define PySpark StructType schema for transactions
   - Fields: txn_id, account_id, amount (Decimal), currency, timestamp, merchant_id, status

2. **Validation Function** (10 points)
   - Create `pipelines/validators/transaction_validator.py`
   - Implement `validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]`
   - Validation rules:
     - txn_id unique
     - amount positive and <= $10,000
     - currency in ['USD', 'EUR', 'GBP']
     - account_id non-null
   - Include comprehensive docstring
   - Add logging

3. **Test Suite** (7 points)
   - Create `tests/validators/test_transaction_validator.py`
   - Test cases:
     - Valid transactions pass
     - Negative amounts rejected
     - Amounts exceeding limit rejected
     - Invalid currency rejected
     - Duplicate txn_id rejected
   - Use `pyspark.testing.utils.assertDataFrameEqual`

4. **Documentation** (3 points)
   - Create `docs/transaction_validator.md`
   - Document validation rules
   - Include usage examples
   - Note PCI-DSS compliance (no card data validation, only metadata)

**Deliverables:**
- All 4 files created
- Screenshot or log of your Claude Code prompts
- Brief reflection (100-200 words): What prompting techniques worked best?

---

## Grading Rubric

### Part A: Multiple Choice (25 points)
- 1 point per correct answer
- No partial credit

### Part B: Practical Exercises (50 points total)

**Exercise 1: settings.json (10 points)**
- Permissions configured correctly (3 pts)
- Environment variables defined (2 pts)
- Hook configured (2 pts)
- Valid JSON syntax (2 pts)
- Default model set (1 pt)

**Exercise 2: CLAUDE.md (10 points)**
- Project overview clear (2 pts)
- Technology stack complete (2 pts)
- Coding standards specified (2 pts)
- Security requirements detailed (2 pts)
- PCI-DSS compliance notes (2 pts)

**Exercise 3: Slash Command (10 points)**
- Correct frontmatter with description (2 pts)
- Argument handling ($1) (2 pts)
- Comprehensive test requirements (3 pts)
- PySpark-specific instructions (2 pts)
- Valid markdown format (1 pt)

**Exercise 4: Agent Definition (10 points)**
- Correct JSON structure (2 pts)
- Clear description (2 pts)
- Detailed compliance prompt (3 pts)
- Tool restrictions appropriate (2 pts)
- Severity rating instruction (1 pt)

**Exercise 5: Effective Prompt (10 points)**
- Clear task statement (3 pts)
- Complete context (schema, rules) (3 pts)
- Specified format (signature, returns) (3 pts)
- Follows Task+Context+Format (1 pt)

### Part C: Hands-On Project (25 points)

**Schema Definition (5 points)**
- Correct StructType usage (2 pts)
- All required fields (2 pts)
- Proper data types (Decimal for amount) (1 pt)

**Validation Function (10 points)**
- Correct function signature (2 pts)
- All validation rules implemented (4 pts)
- Returns tuple of DataFrames (2 pts)
- Logging included (1 pt)
- Type hints and docstring (1 pt)

**Test Suite (7 points)**
- All 5 test cases present (5 pts)
- Uses assertDataFrameEqual (1 pt)
- Tests pass (1 pt)

**Documentation (3 points)**
- Clear explanation (2 pts)
- Usage examples (1 pt)

---

## Answer Key

### Part A: Multiple Choice Answers

1. **B** - To assist developers with AI-powered coding in the terminal
2. **D** - Both B and C (`--plan` and `--permission-mode plan`)
3. **C** - Sonnet
4. **C** - Glob
5. **B** - Ctrl+D
6. **D** - Read-only with `allow: ["Read", "Grep", "Glob"]`
7. **B** - `.claude/settings.local.json`
8. **A** - Performed manually by developers
9. **C** - PCI-DSS
10. **B** - Grant only necessary permissions for the task
11. **B** - To provide persistent project context and standards
12. **B** - `.claude/CLAUDE.md` or root directory (project-level)
13. **B** - Coding standards, PySpark patterns, compliance requirements
14. **B** - User > Project > System
15. **B** - PCI-DSS requirements (no CVV storage, tokenization, masking)
16. **A** - `.claude/commands/`
17. **B** - The first argument passed to the command
18. **B** - Markdown file with frontmatter and prompt
19. **C** - `> /security-review`
20. **B** - `"requireApproval": ["Edit", "Write"]`
21. **B** - To provide specialized, focused AI assistants for specific tasks
22. **C** - At specific points in the workflow (PreToolUse, PostToolUse, etc.)
23. **A** - Model Context Protocol
24. **B** - Task + Context + Format
25. **B** - When exploring, analyzing, or debugging without making changes

### Part B: Sample Solutions

**Exercise 1: settings.json**

```json
{
  "$schema": "https://claude.ai/schemas/settings.json",
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  },
  "defaultModel": "sonnet",
  "env": {
    "SPARK_ENV": "development",
    "PYSPARK_PYTHON": "python3"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/hooks/audit-log.py",
            "description": "Audit all tool usage"
          }
        ]
      }
    ]
  }
}
```

**Exercise 2: CLAUDE.md**

```markdown
# Payment Processing Pipeline Project

## Project Overview
Banking payment processing data pipelines for transaction validation, fraud detection, and settlement processing.

## Technology Stack
- **Language**: Python 3.10+
- **Framework**: PySpark 3.5+
- **Storage**: Delta Lake 3.0+
- **Testing**: pytest, pytest-spark
- **Code Quality**: Ruff, Black, mypy

## Coding Standards

### Python Style
- Follow PEP 8 style guide
- Use type hints for all function signatures
- Docstrings required (Google style)
- 4 spaces for indentation

### PySpark Best Practices
- Use DataFrame API (not RDD API)
- Avoid `collect()` on large datasets
- Use `broadcast()` for small lookup tables (<10MB)
- Define explicit schemas with StructType
- Partition appropriately (128MB target per partition)

## Security Requirements
- **Mask PII in logs**: Account numbers show last 4 digits only
- **Use Decimal for money**: Never use Float/Double for amounts
- **Parameterized queries**: No string concatenation in SQL
- **Environment variables**: All secrets from env vars, never hardcoded

### Example: Masking Account Numbers
```python
# Good: Masked logging
logger.info(f"Processing account: ****{account_id[-4:]}")

# Bad: Exposing PII
logger.info(f"Processing account: {account_id}")  # NEVER!
```

## PCI-DSS Compliance

**NEVER:**
- ❌ Store CVV/CVV2 codes
- ❌ Log credit card numbers
- ❌ Include card data in error messages

**ALWAYS:**
- ✅ Tokenize card data before storage
- ✅ Encrypt cardholder data (AES-256)
- ✅ Audit all access to payment data
- ✅ Mask card numbers (show last 4 only)

## Testing
- Minimum coverage: 80%
- Use sample data (never real card numbers)
- Test data: Visa test card `4111111111111111`
```

**Exercise 3: generate-tests.md**

```markdown
---
description: Generate pytest tests for PySpark validation functions - Usage: /generate-tests <file>
---

Generate comprehensive pytest test cases for: $1

Test Requirements:

## Happy Path
- Valid data passes all validation rules
- Use sample DataFrames with 3-5 records
- Assert using `pyspark.testing.utils.assertDataFrameEqual`

## Edge Cases
- Null values in required fields
- Empty DataFrames
- Schema mismatches (extra/missing columns)
- Boundary values (e.g., amount = 0.01, amount = 10000.00)

## Error Conditions
- Negative amounts
- Invalid currency codes
- Duplicate transaction IDs
- Missing required fields

## Banking-Specific Scenarios
- Overdraft attempts (balance insufficient)
- Duplicate transaction detection (idempotency)
- Amount exceeding daily limits
- Invalid account status transitions

Test Structure:
```python
import pytest
from pyspark.sql import SparkSession
from pyspark.testing.utils import assertDataFrameEqual

@pytest.fixture(scope="session")
def spark():
    return SparkSession.builder.master("local[2]").getOrCreate()

def test_[scenario_name](spark):
    # Arrange
    input_data = [...]
    input_df = spark.createDataFrame(input_data, schema)

    # Act
    result_df = function_to_test(input_df)

    # Assert
    expected_df = spark.createDataFrame(expected_data, schema)
    assertDataFrameEqual(result_df, expected_df)
```

Target: 80% code coverage
Framework: pytest with pyspark.testing.utils
```

**Exercise 4: Agent Definition**

```json
{
  "agents": {
    "compliance-checker": {
      "description": "PCI-DSS compliance auditor for data pipelines",
      "prompt": "You are a PCI-DSS compliance expert for banking data pipelines. Audit PySpark code for compliance violations:\n\n**Check for:**\n1. CVV/CVV2 storage (NEVER allowed - PCI-DSS 3.2.2)\n2. Unencrypted card data in DataFrames or Delta Lake tables\n3. Credit card numbers in logs, error messages, or .show() output\n4. Missing tokenization for card data\n5. Improper data masking in non-production environments\n\n**For each violation:**\n- Provide file:line reference\n- Severity: Critical, High, Medium, Low\n- Explain PCI-DSS requirement violated\n- Suggest remediation with code example\n\n**Output format:**\n```\nVIOLATION: [Description]\nFile: [path:line]\nSeverity: [Critical/High/Medium/Low]\nPCI-DSS Requirement: [section number]\nRemediation: [specific fix with code]\n```",
      "tools": ["Read", "Grep", "Glob"]
    }
  }
}
```

**Exercise 5: Effective Prompt**

```
> Generate a PySpark transaction validation function

TASK: Create a validation function for banking transaction DataFrames

CONTEXT:
Input schema:
```python
from pyspark.sql.types import StructType, StructField, StringType, DecimalType, TimestampType

transaction_schema = StructType([
    StructField("txn_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),
    StructField("currency", StringType(), nullable=False),
    StructField("timestamp", TimestampType(), nullable=False)
])
```

Validation rules:
- amount must be positive (> 0)
- amount must not exceed $10,000 (daily transaction limit)
- currency must be one of: USD, EUR, GBP
- txn_id must be unique (no duplicates in DataFrame)

Technology: PySpark 3.5+

FORMAT:
Function signature:
```python
from pyspark.sql import DataFrame
from typing import Tuple

def validate_transactions(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """Validate banking transactions for data quality and business rules.

    Returns:
        Tuple of (valid_df, invalid_df)
    """
```

Requirements:
- Include type hints and comprehensive docstring
- Return tuple of (valid_df, invalid_df)
- Add logging for validation failures (use Python logging module)
- Use only PySpark DataFrame operations (no collect(), no pandas)
- Log validation statistics (count of valid vs invalid records)
```

---

## Self-Assessment & Next Steps

### Calculate Your Score

| Section | Your Score | Max Points |
|---------|-----------|------------|
| Part A: Multiple Choice | ___ / 25 | 25 |
| Part B: Exercise 1 | ___ / 10 | 10 |
| Part B: Exercise 2 | ___ / 10 | 10 |
| Part B: Exercise 3 | ___ / 10 | 10 |
| Part B: Exercise 4 | ___ / 10 | 10 |
| Part B: Exercise 5 | ___ / 10 | 10 |
| Part C: Project | ___ / 25 | 25 |
| **TOTAL** | **___ / 100** | **100** |

### Performance Levels

**🏆 Expert (95-100 points):**
- Outstanding mastery of Claude Code
- Ready for advanced Phase 2 and 3
- Consider mentoring others

**✅ Proficient (85-94 points):**
- Strong understanding of concepts
- Ready for Phase 2
- Minor review recommended in weak areas

**📚 Pass (70-84 points):**
- Adequate understanding
- Can proceed to Phase 2 with caution
- Review areas where you scored low

**❌ Needs Review (Below 70 points):**
- Significant gaps in knowledge
- **DO NOT proceed to Phase 2 yet**
- Review Phase 1 sections where you struggled
- Retake assessment after review

### Areas to Review

If you scored poorly in:

**Part A (Questions 1-5): Fundamentals**
→ Review [Phase 1.1: Introduction](./01-introduction-getting-started.md) and [Phase 1.2: Installation](./02-installation.md)

**Part A (Questions 6-10): Security**
→ Review [Phase 1.3.2: Security & Permissions](./03-02-security-permissions.md)

**Part A (Questions 11-15): Memory**
→ Review [Phase 1.3.3: Memory & Context](./03-03-memory-context.md)

**Part A (Questions 16-20): Configuration**
→ Review [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) and [Phase 1.3.6: Slash Commands](./03-06-slash-commands.md)

**Part A (Questions 21-25): Advanced**
→ Review [Phase 1.3.7: Agents](./03-07-agents-subagents.md), [Phase 1.3.8: Hooks](./03-08-hooks-automation.md), [Phase 1.4.2: Basic Prompts](./04-02-basic-prompt-structure.md)

**Part B: Practical Exercises**
→ Practice more with Claude Code in your environment

**Part C: Hands-On Project**
→ Complete more practice projects using prompt engineering techniques

---

## Congratulations!

If you passed this assessment, you've successfully completed **Phase 1: Onboarding**!

### What You've Mastered

✅ **Claude Code Fundamentals**
- Installation and setup
- CLI commands and modes
- Permission system

✅ **Core Concepts**
- Architecture and workflow
- Security and compliance
- Memory and context management
- Project configuration
- Slash commands and agents
- Hooks and automation
- MCP integration
- Git workflows (manual policy)
- Standards and best practices

✅ **Prompt Engineering**
- Task + Context + Format formula
- Effective prompting techniques
- Data engineering specific prompts

### Next Steps

👉 **[Proceed to Phase 2: Data Engineering Build](../phase-2-build/README.md)**

Phase 2 covers:
- BRD to DDT workflow
- Data mapping and schema design
- Code generation from specifications
- Testing strategies
- CI/CD integration
- Real-world coding challenge

---

**Completion Certificate**

```
╔════════════════════════════════════════════════════════╗
║                                                        ║
║         CLAUDE CODE MASTERY - PHASE 1                  ║
║                                                        ║
║  [Your Name]                                           ║
║  has successfully completed Phase 1: Onboarding        ║
║                                                        ║
║  Score: [Your Score] / 100                             ║
║  Level: [Expert/Proficient/Pass]                       ║
║                                                        ║
║  Date: [Today's Date]                                  ║
║                                                        ║
║  Banking IT Data Chapter                               ║
║  Claude Code for PySpark Data Engineering              ║
║                                                        ║
╚════════════════════════════════════════════════════════╝
```

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Assessment Type:** Phase 1 Comprehensive
