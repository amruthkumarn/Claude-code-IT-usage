# Phase 1.3.6: Slash Commands

**Learning Objectives:**
- Understand what slash commands are and their benefits
- Learn built-in slash commands
- Create custom slash commands
- Use command arguments effectively
- Implement banking IT command examples

**Time Commitment:** 45 minutes

**Prerequisites:** Phase 1.3.1-1.3.5 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** Create and use your first custom slash command.

### Try This Right Now

```bash
# 1. Create commands directory
mkdir -p .claude/commands
cd .claude/commands

# 2. Create your first command
cat > hello.md << 'EOF'
---
description: Say hello in a banking professional manner
---

Respond with a professional banking greeting and offer to help with:
- PySpark pipeline development
- Data validation
- Banking compliance checks
EOF

# 3. Test it
claude
> /hello
```

**What you'll see:**
Claude responds with a professional greeting and lists how it can help!

**What just happened?**
- Created a custom slash command in `.claude/commands/`
- Command name = filename (hello.md → /hello)
- Markdown content = the prompt Claude will execute

**Next:** Let's learn to create advanced commands with arguments...

---

## Table of Contents
1. [What are Slash Commands?](#what-are-slash-commands)
2. [Built-in Slash Commands](#built-in-slash-commands)
3. [Creating Custom Commands](#creating-custom-commands)
4. [Command Arguments](#command-arguments)
5. [Banking IT Command Examples](#banking-it-command-examples)
6. [Best Practices](#best-practices)

---

## What are Slash Commands?

**Slash commands** are special commands starting with `/` that trigger predefined actions or prompts in Claude Code.

### Types of Slash Commands

1. **Built-in Commands** - Provided by Claude Code (`/help`, `/clear`, etc.)
2. **Custom Project Commands** - Team-shared (`.claude/commands/`)
3. **Custom User Commands** - Personal (`~/.claude/commands/`)
4. **Plugin Commands** - From installed plugins
5. **MCP Commands** - From MCP servers

### Commands vs Regular Prompts

**Regular prompt:**
```
> Review this code for security issues and provide a detailed report
```

**Slash command:**
```
> /security-review
```

Both can achieve the same result, but commands are:
- Faster to type
- Consistent across sessions
- Shareable with team
- Can include complex logic

---

## Built-in Slash Commands

### Essential Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/clear` | Clear conversation history |
| `/exit` | Exit Claude Code |
| `/model [model]` | Change AI model (sonnet/opus/haiku) |
| `/config` | Open settings |
| `/login` | Login or check auth status |
| `/memory` | Edit memory files |
| `/review` | Request code review |
| `/output-style [style]` | Change output style |

### Usage Examples

```bash
# Get help
> /help

# Change model
> /model opus

# Clear history
> /clear

# Code review
> /review
```

---

## Creating Custom Commands

Custom commands are markdown files in `.claude/commands/` or `~/.claude/commands/`.

### Basic Command Structure

Create `.claude/commands/review.md`:

```markdown
---
description: Perform comprehensive code review
---

Review the code in this project for:
- Security vulnerabilities
- Performance issues
- Code quality problems
- Best practice violations
- Potential bugs

Provide a detailed report with specific file names and line numbers.
```

**Usage:**
```bash
> /review
```

### Command with Arguments

Create `.claude/commands/test.md`:

```markdown
---
description: Generate tests for a file
---

Generate comprehensive unit tests for: $1

Include:
- Happy path tests
- Edge cases
- Error conditions
- Mock external dependencies

Use pytest testing framework with PySpark test utilities.
```

**Usage:**
```bash
> /test pipelines/payment_processing.py
# $1 = "pipelines/payment_processing.py"
```

### Command with Multiple Arguments

Create `.claude/commands/refactor.md`:

```markdown
---
description: Refactor code with specific pattern
---

Refactor the file $1 to use $2 pattern.

Target file: $1
Pattern: $2
Additional notes: $3

Ensure:
- Behavior remains unchanged
- Tests still pass
- Code is more maintainable
```

**Usage:**
```bash
> /refactor pipelines/payment_processing.py "DataFrame API" "remove RDD operations"
# $1 = pipelines/payment_processing.py
# $2 = DataFrame API
# $3 = remove RDD operations
```

### Command with All Arguments

Use `$ARGUMENTS` to get all arguments as a single string:

Create `.claude/commands/explain.md`:

```markdown
---
description: Explain a concept
---

Explain the following concept in the context of this codebase:

$ARGUMENTS

Provide:
1. Clear definition
2. How it's used in this project
3. Code examples from the codebase
4. Best practices
```

**Usage:**
```bash
> /explain window functions and why we use them in our pipelines
# $ARGUMENTS = "window functions and why we use them in our pipelines"
```

---

## Command Arguments

### Argument Syntax

| Syntax | Description | Example |
|--------|-------------|---------|
| `$1` | First argument | `/cmd arg1` → `$1 = "arg1"` |
| `$2` | Second argument | `/cmd arg1 arg2` → `$2 = "arg2"` |
| `$3` | Third argument | `/cmd a b c` → `$3 = "c"` |
| `$ARGUMENTS` | All arguments as string | `/cmd hello world` → `$ARGUMENTS = "hello world"` |

---

## 🔨 Exercise 1: Create Banking Slash Commands (20 minutes)

**Goal:** Build a library of custom commands for banking data engineering.

### Step 1: Set up commands directory

```bash
cd ~/banking-pipeline-project
mkdir -p .claude/commands
cd .claude/commands
```

### Step 2: Create `/test` command (with arguments)

```bash
cat > test.md << 'EOF'
---
description: Generate pytest tests for a file - Usage: /test <filepath>
---

Generate comprehensive pytest tests for: $1

Include test cases for:
1. **Happy Path**: Valid inputs, expected outputs
2. **Edge Cases**: Empty data, null values, boundary conditions
3. **Error Conditions**: Invalid inputs, missing fields, type mismatches
4. **Banking Scenarios**:
   - Overdraft conditions
   - Duplicate transactions
   - Invalid account numbers
   - Amount validation (positive, within limits)

Requirements:
- Use pytest framework
- Use SparkSession fixtures for PySpark tests
- Follow Arrange-Act-Assert pattern
- Include descriptive test names
- Add docstrings explaining what each test validates

Save tests to: tests/test_${1}
EOF
```

**Test it:**
```bash
claude
> /test pipelines/transaction_validator.py
```

### Step 3: Create `/security-review` command

```bash
cat > security-review.md << 'EOF'
---
description: Perform banking security review on code
---

Perform comprehensive security review on: ${1:-.}

## Authentication & Authorization
- [ ] Are credentials properly secured (no hardcoded passwords)?
- [ ] Is service principal auth configured correctly?
- [ ] Are API keys stored in Key Vault or env vars?

## PII & Data Protection
- [ ] Is PII data properly masked in logs?
- [ ] Are account numbers hashed/encrypted?
- [ ] Is sensitive data encrypted at rest and in transit?

## Input Validation
- [ ] Are all external inputs validated?
- [ ] Is there SQL injection prevention?
- [ ] Are DataFrame schemas validated?

## Banking Compliance
- [ ] PCI-DSS: No CVV storage?
- [ ] SOX: Audit trail present?
- [ ] GDPR: Personal data handling compliant?

## Secrets Detection
- [ ] No API keys in code?
- [ ] No database passwords hardcoded?
- [ ] No AWS/Azure credentials in code?

Provide findings with:
- File path and line number
- Severity (Critical/High/Medium/Low)
- Remediation steps
- Code examples for fixes
EOF
```

**Test it:**
```bash
> /security-review
# Reviews entire project

> /security-review pipelines/
# Reviews specific directory
```

### Step 4: Create `/generate-pipeline` command

```bash
cat > generate-pipeline.md << 'EOF'
---
description: Generate a PySpark data pipeline - Usage: /generate-pipeline <name>
---

Generate a PySpark data pipeline called: $1

Follow our banking project standards:

## File Structure
```
pipelines/$1/
├── __init__.py
├── main.py           # Main pipeline entry point
├── transformations.py # Business logic transformations
├── validations.py     # Data quality validations
└── config.py         # Pipeline configuration
```

## Code Requirements
1. **Explicit Schema**: Use StructType (no schema inference)
2. **Type Hints**: All functions must have type annotations
3. **Decimal for Money**: Use Decimal(18,2) for all monetary amounts
4. **Error Handling**: Try/except with proper logging
5. **Data Quality**: Validate nulls, duplicates, business rules
6. **Audit Logging**: Log all data transformations
7. **PII Masking**: Mask sensitive fields in logs

## Pipeline Template
```python
from decimal import Decimal
from typing import Tuple
from pyspark.sql import DataFrame, SparkSession
from pyspark.sql.types import StructType, StructField, StringType, DecimalType
import logging

logger = logging.getLogger(__name__)

def read_source(spark: SparkSession, path: str) -> DataFrame:
    """Read source data with explicit schema."""
    schema = StructType([
        StructField("account_id", StringType(), False),
        StructField("amount", DecimalType(18, 2), False),
        # Add more fields
    ])
    return spark.read.schema(schema).parquet(path)

def validate(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """Validate data quality."""
    # Validation logic
    pass

def transform(df: DataFrame) -> DataFrame:
    """Apply business transformations."""
    # Transformation logic
    pass

def write_output(df: DataFrame, path: str) -> None:
    """Write to Delta Lake."""
    df.write.format("delta").mode("overwrite").save(path)
```

Generate complete pipeline code following this template.
EOF
```

**Test it:**
```bash
> /generate-pipeline payment_processing
```

### Step 5: Test all your commands

```bash
claude
> /help
# You should see your custom commands listed!

> /test pipelines/validator.py
> /security-review
> /generate-pipeline customer_enrichment
```

### ✅ Checkpoint
- [ ] Created `/test` command with file argument
- [ ] Created `/security-review` command for compliance
- [ ] Created `/generate-pipeline` command with template
- [ ] Tested all commands successfully
- [ ] Commands appear in `/help` output

### 💻 Terminal Session Example

```bash
$ claude

> /help

Available commands:
  Built-in:
    /help              Show this help
    /config            View configuration
    /clear             Clear conversation

  Custom:
    /test <file>       Generate pytest tests
    /security-review   Perform security audit
    /generate-pipeline Generate PySpark pipeline

> /test pipelines/transaction_validator.py

🤖 I'll generate comprehensive pytest tests for transaction_validator.py

🔧 Tool Use: Write
File: tests/test_transaction_validator.py

```python
import pytest
from decimal import Decimal
from pyspark.sql import SparkSession
from pipelines.transaction_validator import validate_transaction_amount

@pytest.fixture(scope="session")
def spark():
    return SparkSession.builder.master("local[2]").getOrCreate()

def test_valid_transaction_amount(spark):
    """Test validation accepts valid positive amounts."""
    # Arrange
    data = [("TXN001", Decimal("100.50"))]
    df = spark.createDataFrame(data, ["txn_id", "amount"])

    # Act
    valid_df, invalid_df = validate_transaction_amount(df)

    # Assert
    assert valid_df.count() == 1
    assert invalid_df.count() == 0

def test_negative_amount_rejected(spark):
    """Test validation rejects negative amounts."""
    # Arrange
    data = [("TXN002", Decimal("-50.00"))]
    df = spark.createDataFrame(data, ["txn_id", "amount"])

    # Act
    valid_df, invalid_df = validate_transaction_amount(df)

    # Assert
    assert valid_df.count() == 0
    assert invalid_df.count() == 1

# ... more tests ...
```

Approve this write? (yes/no) yes

✅ Created: tests/test_transaction_validator.py with 8 test cases
```

### 🎯 Challenge: Create More Commands

Try creating these additional commands:
1. `/docs` - Generate documentation for a pipeline
2. `/optimize` - Suggest performance optimizations
3. `/compliance` - Check PCI-DSS/SOX/GDPR compliance

<details>
<parameter name="summary">💡 Example: /docs command

```bash
# Without quotes (space separates arguments)
> /refactor pipelines/payment_processing.py window_functions
# $1 = "pipelines/payment_processing.py"
# $2 = "window_functions"

# With quotes (treat as single argument)
> /refactor "pipelines/payment processing.py" "window function optimization"
# $1 = "pipelines/payment processing.py"
# $2 = "window function optimization"
```

---

## Banking IT Command Examples

### Example 1: Compliance Check

`.claude/commands/compliance-check.md`:

```markdown
---
description: Check code for banking compliance (PCI-DSS, SOX, GDPR)
---

Perform a comprehensive compliance check on: ${1:-.}

Check for:

## PCI-DSS Compliance
- Are credit card numbers being logged in DataFrame operations?
- Is sensitive data encrypted at rest and in transit?
- Are CVV codes being stored (must NOT be stored)?
- Is tokenization used for card data?

## SOX Compliance
- Are all financial transactions auditable?
- Is there proper separation of duties in pipeline code?
- Are data transformations logged?

## GDPR Compliance
- Is personal data properly classified (PII fields)?
- Is there a data retention policy in pipelines?
- Can users request data deletion?
- Is consent properly tracked?

## General Security
- SQL injection vulnerabilities in dynamic SQL?
- Hardcoded credentials in Spark configs?
- Proper authentication/authorization for data access?
- PII data properly masked/encrypted?

Provide a detailed report with:
- File paths and line numbers
- Severity (Critical/High/Medium/Low)
- Remediation steps
```

**Usage:**
```bash
> /compliance-check
# Checks entire project

> /compliance-check pipelines/validators/
# Checks specific directory
```

### Example 2: Security Review

`.claude/commands/security-review.md`:

```markdown
---
description: Comprehensive security review
---

Perform security analysis on: ${1:-.}

## Authentication & Authorization
- Are data sources properly protected?
- Is service principal validation correct?
- Are permissions checked before data access?
- Secure credential management (Key Vault)?

## Input Validation
- All external data validated?
- SQL injection prevention in dynamic queries?
- Data schema validation?
- Column-level security enforced?

## Data Protection
- Sensitive data encrypted (at-rest/in-transit)?
- Secrets management proper (Key Vault, environment vars)?
- PII data hashing/masking implemented?
- Encryption keys rotated?

## Logging & Monitoring
- Security events logged?
- PII excluded from logs?
- Error messages don't leak sensitive info?
- Audit trails for data access?

## Dependencies
- Known vulnerabilities in Python packages?
- Outdated packages (requirements.txt)?
- Suspicious dependencies?

Provide actionable recommendations with code examples.
```

### Example 3: Generate Pipeline Documentation

`.claude/commands/pipeline-docs.md`:

```markdown
---
description: Generate data pipeline documentation
---

Generate data pipeline documentation for: $1

Include:
- Pipeline inputs and outputs
- Data schemas (StructType definitions)
- Transformation steps
- Data quality checks
- Error handling
- Performance considerations

Format as comprehensive markdown documentation.

Banking-specific requirements:
- Document all validation rules
- Include audit logging notes
- Specify compliance requirements (PCI-DSS, GDPR)
- Note data retention policies
- Document PII handling
```

**Usage:**
```bash
> /pipeline-docs pipelines/payment_processing.py
```

### Example 4: Test Coverage Report

`.claude/commands/coverage.md`:

```markdown
---
description: Analyze test coverage and generate missing tests
---

Analyze test coverage for: $1

1. Identify all functions/transformations
2. Check which have tests
3. Calculate coverage percentage
4. Generate tests for untested code

Focus on:
- Edge cases
- Error conditions
- Banking-specific scenarios (overdraft, insufficient funds, etc.)
- Data quality validations
- Schema evolution scenarios
- Null handling

Generate pytest tests following our standards:
- Descriptive test names
- Arrange-Act-Assert pattern
- Use SparkSession fixtures
- Test data isolation
- Sample DataFrames for testing
```

### Example 5: Release Notes Generator

`.claude/commands/release-notes.md`:

```markdown
---
description: Generate release notes from git commits
---

Generate release notes for version: $1

Review git commits since last release and create:

## Features
- New functionality added

## Improvements
- Enhancements to existing features

## Bug Fixes
- Issues resolved

## Security
- Security patches and updates

## Breaking Changes
- Changes requiring migration

## Database Changes
- Schema migrations required

Format for:
- Internal team (technical details)
- Stakeholders (business impact)
- Compliance team (audit information)
```

**Usage:**
```bash
> /release-notes v2.5.0
```

---

## Best Practices

### 1. Descriptive Names

**Good:**
```
/security-review
/generate-tests
/compliance-check
```

**Bad:**
```
/sr
/gt
/check
```

### 2. Clear Descriptions

```markdown
---
description: Generate unit tests for a specific file
---
```

Not:
```markdown
---
description: Tests
---
```

### 3. Document Arguments

```markdown
---
description: Refactor code - Usage: /refactor <file> <pattern>
---

Refactor $1 to use $2 pattern.

Arguments:
- $1: File path to refactor
- $2: Pattern to apply (e.g., "DataFrame API", "window-functions")
```

### 4. Provide Context

```markdown
---
description: Generate data pipeline
---

Generate a new data pipeline for: $ARGUMENTS

Use our project standards:
- PySpark 3.5+
- StructType for schema definitions
- Error handling with try/except
- Logging with Python logging module

Follow the pattern in pipelines/example_pipeline.py
```

### 5. Include Examples

```markdown
---
description: Database query optimization
---

Optimize the database queries in: $1

Examples of optimizations:
1. Add indexes for frequently queried columns
2. Use EXPLAIN ANALYZE to identify slow queries
3. Implement query result caching
4. Batch multiple queries
5. Use connection pooling

Provide before/after performance estimates.
```

### 6. Team Organization

```
.claude/commands/
├── README.md                  # Command documentation
├── dev/                       # Development commands
│   ├── test.md
│   ├── lint.md
│   └── format.md
├── review/                    # Code review commands
│   ├── security-review.md
│   ├── compliance-check.md
│   └── performance-review.md
├── generate/                  # Code generation
│   ├── pipeline.md
│   ├── test-suite.md
│   └── migration.md
└── docs/                      # Documentation
    ├── pipeline-docs.md
    └── release-notes.md
```

### 7. Version Control

```bash
# Execute these commands manually (banking IT policy)
# Commit custom commands
git add .claude/commands/
git commit -m "docs: add custom Claude Code commands for team"

# Document in README
echo "## Claude Code Commands" >> README.md
echo "Run \`/help\` to see available custom commands" >> README.md
```

**Note**: Per banking IT policy, execute all git commands manually. Claude Code does NOT run git operations.

---

## Summary

In this subsection, you learned:

### Core Concepts
- ✅ Slash commands for quick, repeatable tasks
- ✅ Built-in vs custom commands
- ✅ Project vs user commands

### Implementation
- ✅ Creating custom commands with markdown
- ✅ Using arguments ($1, $2, $ARGUMENTS)
- ✅ Frontmatter for metadata

### Banking Applications
- ✅ Compliance checking commands
- ✅ Security review automation
- ✅ Pipeline documentation generation
- ✅ Test coverage analysis
- ✅ Release notes automation

---

## Next Steps

👉 **[Continue to 1.3.7: Agents & Sub-agents](./03-07-agents-subagents.md)**

**Quick Practice:**
1. Create `/security-review` command
2. Create `/generate-tests` command with arguments
3. Test your commands in Claude Code

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Settings
- [Phase 1.3.7: Agents & Sub-agents](./03-07-agents-subagents.md) - Specialized agents
- [Phase 1.3.12: Templates Library](./03-12-templates-library.md) - Command templates

---

**Last Updated:** 2025-10-24
**Version:** 1.0
