# Section 14: Templates Library

## Table of Contents
1. [Settings Templates](#settings-templates)
2. [Memory Templates](#memory-templates)
3. [Slash Command Templates](#slash-command-templates)
4. [Hook Script Templates](#hook-script-templates)
5. [Agent Configuration Templates](#agent-configuration-templates)

---

## Settings Templates

### Template 1: Standard Development

`.claude/settings.json`:
```json
{
  "$schema": "https://claude.ai/schemas/settings.json",
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Task", "TodoWrite"],
    "requireApproval": ["Edit", "Write", "Bash"],
    "deny": ["WebSearch"]
  },
  "defaultModel": "sonnet",
  "env": {
    "SPARK_ENV": "development",
    "PYSPARK_PYTHON": "python3"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "./.claude/hooks/lint-check.sh"
        }]
      }
    ]
  }
}
```

### Template 2: Production Read-Only

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch"]
  },
  "defaultModel": "opus",
  "hooks": {
    "PreToolUse": [{
      "matcher": ".*",
      "hooks": [{
        "type": "command",
        "command": "./.claude/hooks/audit-log.sh"
      }]
    }]
  }
}
```

### Template 3: Security Audit

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": "*"
  },
  "defaultModel": "opus",
  "outputStyle": "security-focused",
  "agents": {
    "security-auditor": {
      "description": "Security vulnerability scanner for data pipelines",
      "prompt": "You are a security expert. Identify vulnerabilities in PySpark data pipelines and Python code.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    }
  }
}
```

---

## Memory Templates

### Template 1: Basic Project Memory

`.claude/CLAUDE.md`:
```markdown
# {{PROJECT_NAME}}

## Overview
{{Brief description of the data pipeline project}}

## Technology Stack
- Language: Python 3.9+
- Framework: PySpark 3.4+
- Storage: {{Delta Lake/Parquet/Iceberg}}
- Database: {{Snowflake/Redshift/BigQuery}}
- Testing: pytest
- Orchestration: {{Airflow/Databricks Workflows}}

## Coding Standards

### Code Style
- Use 4 spaces for indentation
- Maximum line length: 100
- Follow PEP 8 style guide
- Use type hints for all functions

### Python Best Practices
- Use context managers for resource handling
- Prefer list comprehensions over map/filter
- Use f-strings for string formatting
- Explicit return types for public functions

### PySpark Best Practices
- Use DataFrame API over RDD API
- Avoid collect() on large datasets
- Use broadcast for small lookup tables
- Partition data appropriately
- Cache intermediate results when reused

## Security Requirements
- All data access requires authentication
- Use parameterized queries for SQL
- Never log PII (account numbers, SSNs, names)
- Encrypt data at rest and in transit
- Validate all input data schemas

## Testing
- Minimum test coverage: 80%
- Write tests before implementation (TDD)
- Mock external data sources
- Use small sample datasets for unit tests

## Git Workflow
- Branch naming: `feature/`, `fix/`, `refactor/`
- Commit format: Conventional Commits
- Require 2 PR approvals for production code
```

### Template 2: Banking Data Engineering Project Memory

`.claude/CLAUDE.md`:
```markdown
# {{BANKING_PROJECT_NAME}}

## Compliance Requirements

### PCI-DSS
NEVER:
- Log credit card numbers (full or partial)
- Store CVV/CVV2 codes
- Store PIN blocks in logs or temporary tables
- Include full card numbers in error messages

ALWAYS:
- Tokenize card numbers before storage
- Encrypt cardholder data (CHD)
- Mask card numbers in logs (show only last 4 digits)
- Audit all access to payment data

### SOX Compliance
- All financial transactions must be auditable
- Maintain complete data lineage
- Change management required for production pipelines
- Separation of duties enforced
- Data integrity checks mandatory

### GDPR
- Personal data encrypted at rest and in transit
- Right to deletion implemented (data purge pipelines)
- Data retention: 7 years for financial records
- Data residency requirements enforced
- Consent tracking for marketing data

## Data Security Standards
- Authentication: Service principals with role-based access
- Encryption: AES-256 for data at rest
- Data masking: Hash or tokenize PII in non-prod environments
- Access logging: All data access logged to audit tables
- Network security: Private endpoints only, no public access

## PySpark Security Patterns
```python
# GOOD: Masked logging
logger.info(f"Processing account: ****{account_id[-4:]}")

# BAD: Exposing PII
logger.info(f"Processing account: {account_id}")

# GOOD: Schema validation
expected_schema = StructType([
    StructField("account_id", StringType(), False),
    StructField("amount", DecimalType(18, 2), False)
])
df = spark.read.schema(expected_schema).parquet(path)

# GOOD: Data quality checks
assert df.filter(col("amount") < 0).count() == 0, "Negative amounts detected"
```

## Code Review Requirements
- Security review required for:
  - Authentication/authorization code
  - Payment data processing
  - SQL queries accessing sensitive tables
  - External API integrations
  - Data masking/tokenization logic
- Automated security scan must pass
- No high/critical vulnerabilities
- Data lineage documented

## Testing Standards
- Unit test coverage: ≥80%
- Integration tests for end-to-end pipelines
- Data quality tests (nulls, ranges, referential integrity)
- Performance tests for large datasets
- Never use real card numbers (use test data: 4111111111111111)
- Mock external services (payment gateways, APIs)

## Pipeline Standards
- Idempotent operations (support reruns)
- Incremental processing where possible
- Partition pruning for performance
- Data quality checks at each stage
- Monitoring and alerting configured
- SLA: 99.9% uptime for critical pipelines
```

---

## Slash Command Templates

### Template 1: Security Review

`.claude/commands/security-review.md`:
```markdown
---
description: Comprehensive security review for data pipelines
---

Perform security analysis on: ${1:-.}

Check for:
## Authentication & Authorization
- Data access properly authenticated?
- Row-level security implemented?
- Service principal permissions minimal?

## Data Protection
- PII properly masked/encrypted?
- Card data tokenized?
- Secrets managed in key vault?
- Data at rest encrypted?

## Input Validation
- Schema validation enforced?
- Data quality checks present?
- SQL injection prevention (parameterized)?

## Logging
- Security events logged to audit table?
- PII excluded from logs?
- Sensitive data properly masked?

## Compliance
- PCI-DSS requirements met?
- SOX audit trail maintained?
- GDPR right-to-deletion supported?

Provide file:line references and remediation steps.
```

### Template 2: Generate Tests

`.claude/commands/generate-tests.md`:
```markdown
---
description: Generate comprehensive pytest tests - Usage: /generate-tests <file>
---

Generate unit tests for: $1

Include:
- Happy path tests
- Edge cases (nulls, empty DataFrames, schema mismatches)
- Error conditions (missing files, invalid data)
- Data quality validations
- Mock Spark session and data sources

Use pytest framework with pyspark.testing.utils.
Follow AAA pattern (Arrange, Act, Assert).

Example structure:
```python
import pytest
from pyspark.sql import SparkSession
from pyspark.testing.utils import assertDataFrameEqual

@pytest.fixture
def spark():
    return SparkSession.builder.master("local[1]").getOrCreate()

def test_transform_happy_path(spark):
    # Arrange
    input_data = [{"id": 1, "amount": 100.0}]
    input_df = spark.createDataFrame(input_data)

    # Act
    result_df = transform_function(input_df)

    # Assert
    expected_data = [{"id": 1, "amount": 100.0, "processed": True}]
    expected_df = spark.createDataFrame(expected_data)
    assertDataFrameEqual(result_df, expected_df)
```

Aim for 80% coverage.
```

### Template 3: Data Pipeline Documentation

`.claude/commands/pipeline-docs.md`:
```markdown
---
description: Generate data pipeline documentation
---

Generate comprehensive documentation for: $1

Include:
## Pipeline Overview
- Purpose and business logic
- Data sources (tables, files, APIs)
- Data destinations (tables, files)
- Schedule/trigger (batch, streaming, event-driven)

## Data Lineage
- Input datasets with schemas
- Transformation logic
- Output datasets with schemas
- Dependencies on other pipelines

## Transformations
- Data quality checks
- Business rules applied
- Aggregations and joins
- Partitioning strategy

## Monitoring
- SLA requirements
- Data quality metrics
- Performance metrics
- Alert conditions

## Security & Compliance
- PII handling
- Encryption methods
- Access controls
- Compliance requirements (PCI-DSS, SOX, GDPR)

## Error Handling
- Retry logic
- Dead letter queue
- Alerting mechanisms

Format as Markdown with code examples.
```

---

## Hook Script Templates

### Template 1: Audit Logging

`.claude/hooks/audit-log.sh`:
```bash
#!/bin/bash
# Audit logging for compliance

TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
USER=$(whoami)
TOOL=${TOOL_NAME:-$1}
FILE=${FILE_PATH:-$2}

LOG_FILE="$HOME/.claude/audit.log"
mkdir -p "$(dirname $LOG_FILE)"

echo "$TIMESTAMP | $USER | $TOOL | $FILE" >> "$LOG_FILE"

# Optional: Send to central logging (e.g., Splunk, DataDog)
# curl -X POST https://audit.company.com/api/logs \
#   -H "Content-Type: application/json" \
#   -d "{\"timestamp\":\"$TIMESTAMP\",\"user\":\"$USER\",\"tool\":\"$TOOL\",\"file\":\"$FILE\"}"

exit 0
```

### Template 2: Secrets Detection

`.claude/hooks/detect-secrets.sh`:
```bash
#!/bin/bash
# Detect hardcoded secrets in Python/PySpark code

FILE=${FILE_PATH:-$1}

# Check for common secret patterns
if grep -qE '(password|secret|api_key|token|connection_string)\s*=\s*["\047][^"\047]{8,}' "$FILE"; then
  echo "WARNING: Potential secret detected in $FILE"
  echo "Use environment variables or key vault instead"
  exit 1
fi

# Check for AWS keys
if grep -qE 'AKIA[0-9A-Z]{16}' "$FILE"; then
  echo "WARNING: AWS Access Key detected in $FILE"
  exit 1
fi

# Check for Azure connection strings
if grep -qE 'AccountKey=[A-Za-z0-9+/=]{88}' "$FILE"; then
  echo "WARNING: Azure Storage Account Key detected in $FILE"
  exit 1
fi

# Check for database connection strings with passwords
if grep -qE '(jdbc|mysql|postgresql|snowflake)://[^:]+:[^@]+@' "$FILE"; then
  echo "WARNING: Database connection string with credentials in $FILE"
  exit 1
fi

exit 0
```

### Template 3: Lint Check

`.claude/hooks/lint-check.sh`:
```bash
#!/bin/bash
# Run linter before changes

FILE=${FILE_PATH:-$1}

if [[ $FILE == *.py ]]; then
  # Run black formatter check
  python -m black --check "$FILE" 2>/dev/null
  if [ $? -ne 0 ]; then
    echo "Code formatting issue detected. Run: black $FILE"
    exit 1
  fi

  # Run flake8 linter
  python -m flake8 "$FILE" --max-line-length=100 --ignore=E203,W503
  if [ $? -ne 0 ]; then
    echo "Linting failed. Fix issues above."
    exit 1
  fi

  # Run mypy type checker
  python -m mypy "$FILE" --ignore-missing-imports
  if [ $? -ne 0 ]; then
    echo "Type checking failed. Add type hints or fix errors."
    exit 1
  fi
fi

echo "Lint check passed"
exit 0
```

---

## Agent Configuration Templates

### Template: Banking Data Engineering Agent Suite

`.claude/settings.json`:
```json
{
  "agents": {
    "compliance-checker": {
      "description": "PCI-DSS, SOX, GDPR compliance auditor for data pipelines",
      "prompt": "You are a banking compliance expert for data engineering. Check for: PCI-DSS violations (CVV storage, unencrypted card data), SOX requirements (audit trails, data lineage), GDPR violations (PII handling, retention). Flag issues with file:line references.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "data-quality-auditor": {
      "description": "Data quality and pipeline reliability specialist",
      "prompt": "You are a data quality expert. Identify: missing null checks, schema validation gaps, referential integrity issues, missing data quality tests, inadequate error handling, performance bottlenecks (collect(), cartesian joins).",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "security-reviewer": {
      "description": "Data security auditor",
      "prompt": "You are a data security expert. Identify: unmasked PII in logs, hardcoded secrets, SQL injection risks, insecure data access patterns, missing encryption, over-permissioned service principals.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "pyspark-optimizer": {
      "description": "PySpark performance optimization specialist",
      "prompt": "You are a PySpark performance expert. Identify: unnecessary collect() calls, missing broadcast hints, poor partitioning, redundant shuffles, missing caching, inefficient joins. Suggest optimizations with code examples.",
      "tools": ["Read", "Grep", "Glob"],
      "model": "opus"
    },
    "test-generator": {
      "description": "Pytest test creation specialist",
      "prompt": "You generate comprehensive pytest tests for PySpark pipelines. Include: happy paths, edge cases (nulls, empty DataFrames), data quality checks, error conditions. Use pyspark.testing.utils and fixtures.",
      "tools": ["Read", "Write", "Grep"],
      "model": "sonnet"
    },
    "doc-generator": {
      "description": "Data pipeline documentation specialist",
      "prompt": "You generate clear pipeline documentation including: data lineage, transformations, SLAs, monitoring, security, compliance. Use Markdown with schema diagrams and code examples.",
      "tools": ["Read", "Write", "Grep"],
      "model": "sonnet"
    }
  }
}
```

---

## Quick Start Templates

### New Data Pipeline Project Setup

```bash
#!/bin/bash
# setup-claude.sh - Initialize Claude Code for data pipeline project

# Create directory structure
mkdir -p .claude/{commands,hooks,output-styles}

# Create settings.json
cat > .claude/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  },
  "defaultModel": "sonnet",
  "env": {
    "SPARK_ENV": "development",
    "PYSPARK_PYTHON": "python3"
  }
}
EOF

# Create CLAUDE.md
cat > .claude/CLAUDE.md << 'EOF'
# {{PROJECT_NAME}}

## Technology Stack
- Python 3.9+
- PySpark 3.4+
- Storage: Delta Lake
- Testing: pytest

## Coding Standards
- Style guide: PEP 8
- Type hints required
- Testing: pytest with 80% coverage

## PySpark Best Practices
- Use DataFrame API (not RDD)
- Avoid collect() on large datasets
- Broadcast small lookup tables
- Partition data appropriately

## Security
- Tokenize PII before logging
- Use key vault for secrets
- Validate input schemas
- Encrypt data at rest

## Compliance
- PCI-DSS: No CVV storage, tokenize cards
- SOX: Maintain audit trails
- GDPR: Support data deletion
EOF

# Create requirements.txt
cat > requirements.txt << 'EOF'
pyspark==3.4.0
pytest==7.4.0
pytest-spark==0.6.0
black==23.3.0
flake8==6.0.0
mypy==1.3.0
EOF

# Create pytest.ini
cat > pytest.ini << 'EOF'
[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
addopts = --verbose --cov=src --cov-report=html
EOF

# Update .gitignore
cat >> .gitignore << 'EOF'
.claude/settings.local.json
*.pyc
__pycache__/
.pytest_cache/
.coverage
htmlcov/
*.egg-info/
dist/
build/
spark-warehouse/
metastore_db/
derby.log
EOF

echo "Claude Code for data engineering initialized!"
echo "Next steps:"
echo "1. Edit .claude/CLAUDE.md with your pipeline standards"
echo "2. Run: pip install -r requirements.txt"
echo "3. Run: claude"
```

---

## Data Pipeline Testing Template

### pytest Configuration for PySpark

`tests/conftest.py`:
```python
import pytest
from pyspark.sql import SparkSession


@pytest.fixture(scope="session")
def spark():
    """Create a Spark session for testing."""
    return (
        SparkSession.builder
        .master("local[2]")
        .appName("pytest-pyspark")
        .config("spark.sql.shuffle.partitions", "2")
        .config("spark.default.parallelism", "2")
        .getOrCreate()
    )


@pytest.fixture
def sample_transactions(spark):
    """Sample transaction data for testing."""
    data = [
        ("ACC001", "2024-01-01", 100.00, "USD", "CREDIT"),
        ("ACC001", "2024-01-02", -50.00, "USD", "DEBIT"),
        ("ACC002", "2024-01-01", 200.00, "USD", "CREDIT"),
    ]
    schema = ["account_id", "date", "amount", "currency", "type"]
    return spark.createDataFrame(data, schema)
```

### Example Unit Test

`tests/test_transformations.py`:
```python
import pytest
from pyspark.testing.utils import assertDataFrameEqual
from pyspark.sql.functions import col, sum as _sum


def test_calculate_account_balance(spark, sample_transactions):
    """Test account balance calculation."""
    # Act
    result = (
        sample_transactions
        .groupBy("account_id")
        .agg(_sum("amount").alias("balance"))
        .orderBy("account_id")
    )

    # Assert
    expected_data = [
        ("ACC001", 50.00),
        ("ACC002", 200.00),
    ]
    expected_df = spark.createDataFrame(expected_data, ["account_id", "balance"])

    assertDataFrameEqual(result, expected_df)


def test_filter_debit_transactions(spark, sample_transactions):
    """Test filtering debit transactions."""
    # Act
    result = sample_transactions.filter(col("type") == "DEBIT")

    # Assert
    assert result.count() == 1
    assert result.first()["amount"] == -50.00
```

---

## Summary

This section provided ready-to-use templates for:

- Settings configurations (dev, production, audit)
- Memory files (basic and banking-specific data engineering)
- Slash commands (security, testing, pipeline docs)
- Hook scripts (audit, secrets, linting for Python)
- Agent configurations (compliance, data quality, security, performance)
- Quick setup scripts for data pipeline projects
- PySpark testing patterns with pytest

Copy and customize these templates for your data engineering projects!

---

## Next Steps

1. **[Continue to Section 15: Troubleshooting & FAQ](./15-troubleshooting-faq.md)** - Common issues and solutions
2. **Copy templates to your project** - Adapt as needed
3. **Share with your team** - Standardize across data pipeline projects

---
