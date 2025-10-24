# Phase 1.3.11: Standards & Best Practices

**Learning Objectives:**
- Establish organizational standards for Claude Code
- Configure project setup best practices
- Implement team collaboration workflows
- Apply security best practices
- Optimize performance and resource usage
- Follow banking-specific standards and compliance

**Time Commitment:** 60 minutes

**Prerequisites:** Phase 1.3.1-1.3.10 completed

---

## Table of Contents
1. [Organizational Standards](#organizational-standards)
2. [Project Setup](#project-setup)
3. [Team Collaboration](#team-collaboration)
4. [Security Best Practices](#security-best-practices)
5. [Performance Optimization](#performance-optimization)
6. [Banking-Specific Standards](#banking-specific-standards)

---

## Organizational Standards

### Naming Conventions

**Custom Slash Commands:**
```
.claude/commands/
├── compliance-check.md          # Clear, descriptive
├── security-review.md            # Hyphenated
├── data-quality-check.md        # Hyphenated, descriptive
├── generate-tests.md            # Verb-noun format
└── pipeline-docs.md             # Domain-specific
```

**Avoid:**
```
.claude/commands/
├── cc.md           # Too cryptic
├── sr.md           # Unclear abbreviation
├── check.md        # Too vague
```

**Memory Files:**
```
CLAUDE.md                    # Main project memory (root or .claude/)
.claude/CLAUDE.md           # Alternative location
~/.claude/CLAUDE.md         # User memory (personal standards)
```

**Configuration Files:**
```
.claude/settings.json        # Team-shared configuration
.claude/settings.local.json  # Personal overrides (gitignored)
```

### File Organization

```
banking-data-pipelines/
├── .claude/
│   ├── settings.json              # Team configuration
│   ├── settings.local.json        # Personal (in .gitignore)
│   ├── CLAUDE.md                  # Project memory and standards
│   ├── commands/                  # Custom slash commands
│   │   ├── README.md              # Command documentation
│   │   ├── review/                # Organized by category
│   │   │   ├── security.md
│   │   │   ├── compliance.md
│   │   │   └── performance.md
│   │   ├── generate/
│   │   │   ├── tests.md
│   │   │   ├── docs.md
│   │   │   └── pipeline.md
│   │   └── quality/
│   │       ├── data-quality.md
│   │       └── code-quality.md
│   ├── hooks/                     # Hook scripts
│   │   ├── README.md
│   │   ├── audit-log.py
│   │   ├── security-check.py
│   │   └── pci-compliance.py
│   └── output-styles/             # Custom output styles
│       └── detailed.md
├── pipelines/                     # PySpark pipelines
│   ├── etl/
│   ├── transformers/
│   └── validators/
├── tests/                         # pytest tests
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── .gitignore                     # Include .claude/settings.local.json
├── pyproject.toml                 # Python project configuration
├── requirements.txt               # Dependencies
└── README.md                      # Document Claude Code usage
```

---

## Project Setup

### Initial Setup Checklist

**1. Create `.claude` directory:**
```bash
mkdir -p .claude/{commands,hooks,output-styles}
mkdir -p .claude/commands/{review,generate,quality}
```

**2. Create `settings.json`:**
```json
{
  "$schema": "https://claude.ai/schemas/settings.json",
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  },
  "defaultModel": "sonnet",
  "env": {
    "SPARK_ENV": "development",
    "PYSPARK_PYTHON": "python3",
    "PYTHONPATH": "${PYTHONPATH}:${PWD}/pipelines"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "python ./.claude/hooks/security-check.py"
        }]
      }
    ]
  }
}
```

**3. Create `CLAUDE.md`:**
```markdown
# Banking Data Pipelines Project

## Overview
Real-time and batch data processing pipelines for banking transactions, accounts, and customer data.

## Technology Stack
- **Language**: Python 3.10+
- **Framework**: PySpark 3.5+
- **Storage**: Delta Lake 3.0+
- **Database**: Snowflake, PostgreSQL
- **Testing**: pytest, pytest-spark
- **Code Quality**: Ruff, Black, mypy
- **Orchestration**: Databricks Workflows

## Coding Standards

### Python Style
- Follow PEP 8 style guide (enforced by Ruff)
- Format with Black (line length: 100)
- Use type hints for all function signatures
- 4 spaces for indentation
- Docstrings for all public functions (Google style)

### PySpark Best Practices
- Use DataFrame API (not RDD API)
- Avoid `collect()` on large datasets
- Use `broadcast()` for small lookup tables (<10MB)
- Partition data appropriately (target: 128MB per partition)
- Cache intermediate DataFrames when reused multiple times
- Prefer built-in functions over UDFs for performance
- Use StructType for explicit schema definitions

## Security Requirements
- All data access requires authentication (service principals)
- Use parameterized queries for all SQL operations
- Never log PII: account numbers, SSNs, customer names
- Encrypt data at rest (AES-256) and in transit (TLS 1.2+)
- Validate all input data schemas with StructType
- Mask sensitive data in non-production environments

## Testing Standards
- Minimum test coverage: 80% (enforced in CI/CD)
- Write tests before implementation (TDD preferred)
- Mock external data sources (S3, Snowflake, Kafka)
- Use small sample datasets for unit tests (<1000 records)
- Integration tests on staging cluster with realistic data volumes

## Git Workflow
- Branch naming: `feature/`, `fix/`, `refactor/`, `perf/`, `test/`
- Commit format: Conventional Commits
- Require 2 PR approvals for production code
- All git operations performed manually (no automation)

## Compliance
- PCI-DSS: Tokenize card data, never store CVV
- SOX: Maintain audit trails for financial transactions
- GDPR: Support right to deletion, data retention policies
```

**4. Update `.gitignore`:**
```gitignore
# Claude Code
.claude/settings.local.json
.claude/.sessions/

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# Virtual environments
venv/
ENV/
env/
.venv

# PySpark
derby.log
metastore_db/
spark-warehouse/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Testing
.pytest_cache/
.coverage
htmlcov/
.mypy_cache/
.ruff_cache/

# Secrets
.env
*.key
*.pem
credentials.json
```

**5. Create `README` section:**
```markdown
## Claude Code Setup

This project uses Claude Code for AI-assisted data engineering development.

### Getting Started
1. Install: `pip install claude-code`
2. Login: `claude /login`
3. Navigate to project: `cd banking-data-pipelines`
4. Start: `claude`

### Custom Commands
Run `/help` to see all available commands:
- `/compliance-check` - Check PCI-DSS/SOX/GDPR compliance
- `/security-review` - Security audit for data pipelines
- `/data-quality-check` - Validate data pipeline quality
- `/generate-tests` - Generate pytest test cases
- `/pipeline-docs` - Generate pipeline documentation

See `.claude/commands/README.md` for detailed documentation.

### Code Quality Tools
- **Ruff**: Linting (`ruff check .`)
- **Black**: Formatting (`black .`)
- **mypy**: Type checking (`mypy pipelines/`)
- **pytest**: Testing (`pytest tests/`)
```

---

## Team Collaboration

### Shared Configuration

**What to Commit to Git:**
- ✅ `.claude/settings.json` - Team settings
- ✅ `.claude/CLAUDE.md` - Project standards
- ✅ `.claude/commands/` - Custom commands
- ✅ `.claude/hooks/` - Hook scripts
- ✅ `.claude/output-styles/` - Custom styles
- ✅ `pyproject.toml` - Python project config
- ✅ `requirements.txt` or `poetry.lock` - Dependencies

**What NOT to Commit:**
- ❌ `.claude/settings.local.json` - Personal settings
- ❌ `.claude/.sessions/` - Session history
- ❌ `venv/` or `.venv/` - Virtual environments
- ❌ `__pycache__/` - Python bytecode
- ❌ `.env` - Environment variables with secrets
- ❌ `credentials.json` - API keys or tokens

### Code Review with Claude

**Reviewer Checklist:**
```markdown
# Data Pipeline Code Review Checklist

Before approving PR:
- [ ] Run `/security-review` on changed files
- [ ] Run `/compliance-check` for PII/card data handling
- [ ] Verify test coverage ≥80% with `pytest --cov`
- [ ] Run `/data-quality-check` for pipeline changes
- [ ] Check for secrets with `/detect-secrets`
- [ ] Verify Ruff linting passes: `ruff check .`
- [ ] Verify Black formatting applied: `black --check .`
- [ ] Verify mypy type checking passes: `mypy pipelines/`
- [ ] Review PySpark best practices (no collect(), proper partitioning)
- [ ] Check data lineage documentation updated
```

### Onboarding New Team Members

**Day 1 - Setup:**
```markdown
1. **Install Python and dependencies**
   ```bash
   python3 --version  # Verify Python 3.10+
   python3 -m venv venv
   source venv/bin/activate  # Unix/Mac
   # or: venv\Scripts\activate  # Windows
   pip install -r requirements.txt
   ```

2. **Install Claude Code**
   ```bash
   pip install claude-code
   claude /login
   ```

3. **Clone project and explore**
   ```bash
   git clone <repository-url>
   cd banking-data-pipelines
   cat .claude/CLAUDE.md  # Read project standards
   ```

4. **Test setup**
   ```bash
   claude
   > /help  # See custom commands
   > Explain the project structure and key PySpark pipelines
   ```
```

**Day 2 - Practice:**
```markdown
1. **Explore in Plan Mode (Read-Only)**
   ```bash
   claude --permission-mode plan
   > Explain the customer aggregation pipeline
   > What data quality checks are implemented?
   > Show me an example of PySpark window functions in this codebase
   ```

2. **Small Task with Supervision**
   ```bash
   claude
   > Help me fix the bug in pipelines/validators/transaction.py:45
   > Add a unit test for the duplicate detection logic
   ```

3. **Code Quality Tools**
   ```bash
   # Run linting
   ruff check .

   # Format code
   black .

   # Type checking
   mypy pipelines/

   # Run tests
   pytest tests/
   ```
```

---

## Security Best Practices

### 1. Principle of Least Privilege

**Development Environment:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  }
}
```

**Production (Read-Only Analysis):**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch"]
  }
}
```

### 2. Secrets Management

**Never Do This:**
```python
# config.py - WRONG!
SNOWFLAKE_PASSWORD = "MyP@ssw0rd123"  # NEVER hardcode secrets!
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"  # NEVER!
KAFKA_API_KEY = "abcd1234efgh5678"  # NEVER!
```

**Always Do This:**
```python
# config.py - CORRECT
import os
from typing import Optional

def get_secret(key: str, default: Optional[str] = None) -> str:
    """Get secret from environment variable.

    Args:
        key: Environment variable name
        default: Default value if not found

    Returns:
        Secret value

    Raises:
        ValueError: If secret not found and no default provided
    """
    value = os.getenv(key, default)
    if value is None:
        raise ValueError(f"Secret {key} not found in environment")
    return value

# Load from environment
SNOWFLAKE_PASSWORD = get_secret("SNOWFLAKE_PASSWORD")
AWS_ACCESS_KEY = get_secret("AWS_ACCESS_KEY")
KAFKA_API_KEY = get_secret("KAFKA_API_KEY")
```

**Settings Configuration:**
```json
{
  "env": {
    "SNOWFLAKE_PASSWORD": "${SNOWFLAKE_PASSWORD}",
    "AWS_ACCESS_KEY": "${AWS_ACCESS_KEY}",
    "KAFKA_API_KEY": "${KAFKA_API_KEY}"
  }
}
```

### 3. Audit Logging

Enable for all environments:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": ".*",
      "hooks": [{
        "type": "command",
        "command": "python ./.claude/hooks/audit-log.py",
        "description": "Log all tool usage for compliance"
      }]
    }]
  }
}
```

### 4. Regular Security Reviews

```bash
# Monthly security audit
claude --permission-mode plan

> @security-auditor perform comprehensive security review of data pipelines
>
> Check for:
> - SQL injection vulnerabilities in dynamic queries
> - Hardcoded secrets and credentials
> - Authentication and authorization issues
> - Sensitive data exposure in logs or DataFrames
> - Insecure data serialization (pickle)
> - Unvalidated external inputs
> - PII data properly masked/encrypted
```

---

## Performance Optimization

### 1. Use Sonnet Model

```bash
# Sonnet is the default and recommended model for data engineering
claude

# Or explicitly specify
claude --model sonnet

# For read-only analysis
claude --permission-mode plan
> Analyze the payment processing pipeline architecture and suggest PySpark optimizations
```

**Note:** Sonnet provides excellent performance for all data engineering tasks including code generation, testing, documentation, and complex analysis.

### 2. Manage Context

**Keep Sessions Focused:**
```bash
# Don't: Context bloat
> Tell me about customer_pipeline.py
> Now transaction_pipeline.py
> Now payment_pipeline.py
> Now fraud_detection_pipeline.py
# ... (loses focus, consumes context window)

# Do: Focused request
> Explain the transaction processing pipelines in pipelines/transactions/
> Include: data flow, transformations, data quality checks
```

**Start Fresh When Needed:**
```bash
# If Claude "forgets" context or makes mistakes
Ctrl+D  # Exit
claude  # New session, clean context

# Then provide focused context
> Read pipelines/transactions/payment_processor.py and explain the retry logic
```

### 3. Use Agents for Subtasks

```bash
# Instead of one long session doing everything:
> @security-auditor review security vulnerabilities
> @test-generator create comprehensive pytest tests
> @doc-writer generate pipeline documentation
> @data-validator check data quality validations
> @pyspark-optimizer suggest performance improvements

# Each agent has clean context and specialized focus
```

### 4. PySpark Performance Tips

**Avoid Collect:**
```python
# Bad: Brings all data to driver
df.collect()  # Crashes on large datasets!

# Good: Process in distributed fashion
df.write.parquet("s3://output/")
```

**Use Broadcast Joins:**
```python
# Bad: Sort-merge join for small table
large_df.join(small_df, "key")

# Good: Broadcast small table
from pyspark.sql.functions import broadcast
large_df.join(broadcast(small_df), "key")
```

**Partition Appropriately:**
```python
# Bad: Too many small partitions
df.repartition(10000)  # Overhead!

# Good: Target 128MB per partition
num_partitions = int(data_size_gb * 1024 / 128)
df.repartition(num_partitions)
```

---

## Banking-Specific Standards

### Code Review Standards

```markdown
# Banking Data Pipeline Code Review Checklist

## Security
- [ ] No hardcoded credentials (check config files, connection strings)
- [ ] SQL queries use parameterized statements (no string concatenation)
- [ ] Authentication required for all data sources (service principals)
- [ ] Input validation implemented with StructType schemas
- [ ] Error messages don't leak sensitive info (no account numbers in exceptions)
- [ ] PII/PCI data properly encrypted (AES-256 at rest, TLS in transit)

## Compliance
- [ ] **PCI-DSS**: No card data in logs or temporary tables
- [ ] **PCI-DSS**: CVV codes NEVER stored anywhere
- [ ] **SOX**: Audit trail for financial transactions (Delta Lake change feed)
- [ ] **GDPR**: Personal data properly handled with encryption and consent
- [ ] **GDPR**: Right to deletion implemented (soft delete flags)
- [ ] Data retention policies enforced (7 years for financial records)

## Data Quality
- [ ] Schema validation for all inputs (StructType enforcement)
- [ ] Null handling implemented (explicit checks, non-nullable fields)
- [ ] Data type validation (Decimal for money, not Float/Double)
- [ ] Duplicate detection logic tested
- [ ] Referential integrity checks (foreign keys validated)
- [ ] Data lineage documented (source → transformations → destination)

## Code Quality (Python/PySpark)
- [ ] Test coverage ≥ 80% (pytest with pytest-cov)
- [ ] Ruff linting passes with no errors (`ruff check .`)
- [ ] Black formatting applied (`black .`)
- [ ] mypy type checking passes (`mypy pipelines/`)
- [ ] No `print()` statements in production code (use logging)
- [ ] Proper logging with appropriate levels (INFO, WARNING, ERROR)
- [ ] Type hints on all function signatures
- [ ] Docstrings for all public functions (Google style)

## PySpark Best Practices
- [ ] DataFrame API used (not RDD API)
- [ ] No `collect()` on large datasets
- [ ] Broadcast joins for small tables (<10MB)
- [ ] Appropriate partitioning strategy (128MB target per partition)
- [ ] Caching used for reused DataFrames
- [ ] Built-in functions preferred over UDFs
- [ ] StructType schemas defined explicitly
```

### Python Code Quality Tools

**pyproject.toml Configuration:**
```toml
[project]
name = "banking-data-pipelines"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = [
    "pyspark>=3.5.0",
    "delta-spark>=3.0.0",
    "pytest>=7.4.0",
    "pytest-spark>=0.6.0",
]

[tool.black]
line-length = 100
target-version = ['py310', 'py311']
include = '\.pyi?$'
exclude = '''
/(
    \.git
  | \.venv
  | build
  | dist
  | spark-warehouse
)/
'''

[tool.ruff]
line-length = 100
target-version = "py310"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
    "S",   # flake8-bandit (security)
]
ignore = []

[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]
"tests/*" = ["S101"]  # Allow assert in tests

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
strict_equality = true

[[tool.mypy.overrides]]
module = "pyspark.*"
ignore_missing_imports = true

[tool.pytest.ini_options]
minversion = "7.0"
addopts = "-ra -q --strict-markers --cov=pipelines --cov-report=html --cov-report=term --cov-fail-under=80"
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]

[tool.coverage.run]
source = ["pipelines"]
omit = ["*/tests/*", "*/test_*.py"]

[tool.coverage.report]
precision = 2
show_missing = true
skip_covered = false
fail_under = 80
```

### Deployment Standards

```markdown
## Pre-Deployment Checklist

### Code Quality
- [ ] All pytest tests passing (`pytest`)
- [ ] Ruff linting passes (`ruff check .`)
- [ ] Black formatting applied (`black --check .`)
- [ ] mypy type checking passes (`mypy pipelines/`)
- [ ] Test coverage ≥ 80% (`pytest --cov --cov-fail-under=80`)

### Security
- [ ] Security scan completed (no high/critical issues)
- [ ] Dependency vulnerability scan (`pip-audit` or `safety check`)
- [ ] Compliance check passed (`/compliance-check`)
- [ ] No secrets in code or logs (`/detect-secrets`)
- [ ] Service principal permissions validated (least privilege)

### Review Process
- [ ] Code review approved (minimum 2 reviewers)
- [ ] Architecture review for major changes (data engineer + architect)
- [ ] Data impact analysis completed (downstream consumers notified)
- [ ] Performance testing completed on staging cluster

### Infrastructure
- [ ] Database migrations tested (DDL scripts validated)
- [ ] Spark cluster configuration verified (memory, cores, autoscaling)
- [ ] Resource allocation validated (cost estimate approved)
- [ ] Rollback plan documented and tested

### Monitoring
- [ ] Monitoring alerts configured (Datadog/Grafana)
- [ ] Data quality checks in place (Great Expectations/Deequ)
- [ ] Performance metrics tracked (throughput, latency, errors)
- [ ] Runbook updated with deployment steps and troubleshooting
- [ ] On-call rotation notified of deployment
```

### PySpark Pipeline Standards

**Example Pipeline with Banking Standards:**

`pipelines/transactions/payment_processor.py`:
```python
"""Payment transaction processing pipeline with PCI-DSS compliance."""

from typing import List, Dict, Any
from decimal import Decimal
from pyspark.sql import SparkSession, DataFrame
from pyspark.sql import functions as F
from pyspark.sql.types import (
    StructType, StructField, StringType, DecimalType, TimestampType
)
import logging

logger = logging.getLogger(__name__)


class PaymentProcessor:
    """Process banking payment transactions with compliance and security controls.

    Implements PCI-DSS requirements for payment data handling:
    - Tokenization of card numbers
    - Masking of sensitive data in logs
    - Audit trail for all transactions
    - Encryption at rest via Delta Lake
    """

    def __init__(self, spark: SparkSession):
        self.spark = spark
        self.schema = self._get_schema()

    @staticmethod
    def _get_schema() -> StructType:
        """Define strict schema for payment transaction data.

        Returns:
            StructType with all required fields and data types
        """
        return StructType([
            StructField("transaction_id", StringType(), nullable=False),
            StructField("account_id", StringType(), nullable=False),
            StructField("amount", DecimalType(18, 2), nullable=False),
            StructField("currency", StringType(), nullable=False),
            StructField("timestamp", TimestampType(), nullable=False),
            StructField("merchant_id", StringType(), nullable=True),
            StructField("card_token", StringType(), nullable=True),  # Tokenized, not raw card
        ])

    def validate_data(self, df: DataFrame) -> DataFrame:
        """Validate data quality and compliance rules.

        Args:
            df: Input DataFrame with payment transactions

        Returns:
            Validated DataFrame (invalid records filtered out)

        Raises:
            ValueError: If validation failure rate exceeds 5%
        """
        total_count = df.count()

        # Check for nulls in required fields
        null_counts = df.select([
            F.count(F.when(F.col(c).isNull(), c)).alias(c)
            for c in ["transaction_id", "account_id", "amount"]
        ]).first()

        # Log validation metrics (no PII)
        logger.info(f"Validation metrics: total={total_count}, nulls={null_counts}")

        # Filter invalid records
        valid_df = df.filter(
            (F.col("transaction_id").isNotNull()) &
            (F.col("account_id").isNotNull()) &
            (F.col("amount") > 0) &
            (F.col("amount") <= 1000000) &  # Max $1M per transaction
            (F.col("currency").isin(["USD", "EUR", "GBP"]))  # Supported currencies
        )

        valid_count = valid_df.count()
        failure_rate = (total_count - valid_count) / total_count if total_count > 0 else 0

        if failure_rate > 0.05:  # Alert if >5% failures
            logger.error(f"High validation failure rate: {failure_rate:.2%}")
            raise ValueError(f"Validation failure rate {failure_rate:.2%} exceeds threshold")

        logger.info(f"Validation complete: {valid_count}/{total_count} records valid")
        return valid_df

    def mask_sensitive_data(self, df: DataFrame) -> DataFrame:
        """Mask PII for compliance (GDPR/PCI-DSS).

        Args:
            df: Input DataFrame with sensitive data

        Returns:
            DataFrame with masked sensitive fields
        """
        return df.withColumn(
            "account_id_masked",
            F.concat(
                F.lit("****"),
                F.substring(F.col("account_id"), -4, 4)
            )
        ).drop("account_id")  # Remove unmasked field

    def process(self, input_path: str, output_path: str) -> None:
        """Execute full payment processing pipeline.

        Args:
            input_path: S3/ADLS path to raw transaction data
            output_path: Delta Lake table path for validated transactions

        Raises:
            Exception: If pipeline fails (logged with details)
        """
        try:
            # Read with strict schema enforcement
            logger.info(f"Reading transactions from {input_path}")
            df = self.spark.read.schema(self.schema).parquet(input_path)

            # Validate data quality
            valid_df = self.validate_data(df)

            # Apply masking for compliance
            processed_df = self.mask_sensitive_data(valid_df)

            # Write to Delta Lake with partitioning
            logger.info(f"Writing validated transactions to {output_path}")
            (
                processed_df.write
                .format("delta")
                .mode("append")  # Idempotent with transaction_id as key
                .partitionBy("timestamp")
                .option("mergeSchema", "false")  # Enforce strict schema
                .save(output_path)
            )

            record_count = processed_df.count()
            logger.info(f"Successfully processed {record_count:,} payment transactions")

        except Exception as e:
            logger.error(f"Payment processing pipeline failed: {str(e)}", exc_info=True)
            raise
```

### Incident Response

```markdown
## Using Claude Code in Production Incidents

### 1. Investigation (Read-Only)
```bash
# ALWAYS use plan mode for production analysis
claude --permission-mode plan

> Analyze the error logs for the payment processing pipeline
> Search for similar PySpark exceptions in the codebase
> Check data quality metrics for anomalies in the last 24 hours
```

### 2. Fix Development (Dev Environment Only)
```bash
# Create hotfix branch
git checkout -b hotfix/payment-processor-null-handling

# Develop fix in dev environment
claude
> Fix the NullPointerException in pipelines/transactions/payment_processor.py
> Add comprehensive pytest tests to prevent regression
> Validate fix with sample payment data

# Review and commit manually
Ctrl+D
git diff
git add pipelines/transactions/payment_processor.py tests/
git commit -m "fix: handle null amounts in payment validation"
git push origin hotfix/payment-processor-null-handling
```

### 3. Documentation
```bash
claude --permission-mode plan

> Generate incident report with:
> - Timeline of events
> - Root cause analysis
> - Impact assessment (records affected, financial impact)
> - Remediation steps taken
> - Lessons learned
> - Preventive measures

> Update runbook with:
> - New troubleshooting steps
> - Monitoring alerts to add
> - Improved error handling patterns
```

**CRITICAL:** Never use Claude Code directly in production during incidents!
Always develop and test fixes in dev/staging environments first.
```

---

## Summary

In this subsection, you learned:

### Organization
- ✅ Naming conventions for commands, files, and configurations
- ✅ Project structure best practices for Python/PySpark projects
- ✅ What to commit vs. gitignore

### Team Collaboration
- ✅ Onboarding process for data engineers
- ✅ Shared configuration with pyproject.toml
- ✅ Code review workflows with Ruff, Black, mypy, and pytest

### Security
- ✅ Least privilege principle for permissions
- ✅ Secrets management with environment variables
- ✅ Audit logging for compliance
- ✅ Regular security reviews

### Performance
- ✅ Model selection (Sonnet for all tasks)
- ✅ Context management strategies
- ✅ Using agents for focused subtasks
- ✅ PySpark performance optimization tips

### Banking Standards
- ✅ Code review checklists for Python/PySpark data pipelines
- ✅ Code quality tools configuration (Ruff, Black, mypy, pytest)
- ✅ Deployment standards and pre-deployment checklists
- ✅ PySpark pipeline patterns with PCI-DSS compliance
- ✅ Incident response guidelines

---

## Next Steps

👉 **[Continue to 1.3.12: Templates Library](./03-12-templates-library.md)**

**Quick Practice:**
1. Set up `.claude/` directory structure in your project
2. Create `CLAUDE.md` with your team's standards
3. Configure `pyproject.toml` with Ruff, Black, mypy, pytest
4. Run code quality tools on existing PySpark pipelines

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Settings configuration
- [Phase 1.3.10: Git Integration](./03-10-git-integration.md) - Git workflows
- [Phase 1.3.12: Templates Library](./03-12-templates-library.md) - Ready-to-use templates

---

**Last Updated:** 2025-10-24
**Version:** 1.0
