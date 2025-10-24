# Phase 1.3.12: Templates Library

**Learning Objectives:**
- Access ready-to-use templates for settings
- Use memory templates for project standards
- Implement slash command templates
- Deploy hook script templates
- Configure agent templates for banking IT

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.3.1-1.3.11 completed

---

## ⚡ Quick Start (5 minutes)

**Goal:** Use ready-made templates right now!

```bash
# 1. Copy template to your project
cp templates/CLAUDE.md.template ~/my-project/CLAUDE.md

# 2. Customize for your project
# Edit CLAUDE.md with your team's standards

# 3. Use it!
cd ~/my-project
claude
> Generate a PySpark function to validate transactions
# Claude follows YOUR standards from CLAUDE.md!
```

**Key Insight:** Templates ensure consistency across your team!

---

## 🔨 Hands-On Exercise: Create Your Team's Template Library (15 minutes)

**Goal:** Build reusable templates for your banking IT team.

**Scenario:** Your team needs standardized project templates to ensure consistency across all PySpark data engineering projects.

### Step 1: Create Templates Directory (2 min)

```bash
# Create team templates repository
mkdir -p ~/team-claude-templates && cd ~/team-claude-templates
mkdir -p {settings,memory,commands,hooks,agents}

# Initialize git for sharing
git init
echo "# Banking IT Claude Code Templates" > README.md
```

**✅ Checkpoint 1:** Templates repository created.

---

### Step 2: Create Settings Template (3 min)

```bash
cat > settings/banking-it-standard.json <<'EOF'
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
    "ENVIRONMENT": "{{ENVIRONMENT}}"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "bash ./.claude/hooks/detect-secrets.sh",
          "description": "Detect hardcoded secrets"
        }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [{
          "type": "command",
          "command": "bash ./.claude/hooks/audit-log.sh",
          "description": "SOX compliance audit log"
        }]
      }
    ]
  }
}
EOF

echo "✅ Settings template created"
```

**✅ Checkpoint 2:** Settings template ready.

---

### Step 3: Create Memory (CLAUDE.md) Template (5 min)

```bash
cat > memory/CLAUDE-banking-template.md <<'EOF'
# {{PROJECT_NAME}}

## Your Role
You are a senior PySpark data engineer working in banking IT.

## Project Information
- **Team**: {{TEAM_NAME}}
- **Domain**: {{DOMAIN}}  (e.g., Payments, Fraud Detection, Customer Data)
- **Tech Stack**: Python 3.10+, PySpark 3.5+, pytest, Delta Lake
- **Compliance**: PCI-DSS, SOX, GDPR

## Banking IT Standards

### Security (PCI-DSS)
- **NEVER** store full credit card numbers (mask: `****-****-****-1234`)
- **NEVER** store CVV codes
- **ALWAYS** mask PII in logs (account_id, SSN, card numbers)
- **USE** read-only database connections for queries
- **ENCRYPT** sensitive data at rest and in transit

### Code Quality
- Type hints required for all functions
- Docstrings required (Google style)
- 100% test coverage goal
- No hardcoded values (use config files)
- PEP 8 compliant

### File Naming
- Pipelines: `{verb}_{noun}.py` (e.g., `validate_transactions.py`)
- Tests: `test_{module}.py`
- Config: `{environment}.yaml` (e.g., `dev.yaml`, `prod.yaml`)

### Git Workflow
- Feature branches: `feature/{ticket}-{description}`
- Commit messages: Conventional Commits format
- All git operations manual (no automation)
- PR required for all changes

## Sample Data Schema

```python
from pyspark.sql.types import *

transaction_schema = StructType([
    StructField("txn_id", StringType(), False),
    StructField("account_id", StringType(), False),
    StructField("amount", DecimalType(18, 2), False),
    StructField("currency", StringType(), False),
    StructField("timestamp", TimestampType(), False),
    StructField("merchant_id", StringType(), True),
    StructField("status", StringType(), False)
])
```

## Common Patterns

### Validation Function Template
```python
from pyspark.sql import DataFrame
from typing import Tuple

def validate_{entity}(df: DataFrame) -> Tuple[DataFrame, DataFrame]:
    """
    Validate {entity} data.

    Args:
        df: Input DataFrame with {entity}_schema

    Returns:
        Tuple of (valid_df, invalid_df)
    """
    # Validation logic here
    pass
```

### Test Template
```python
import pytest
from pyspark.sql import SparkSession

def test_{function_name}(spark: SparkSession):
    """Test {function_name} with valid data."""
    # Arrange
    # Act
    # Assert
    pass
```
EOF

echo "✅ Memory template created"
```

**✅ Checkpoint 3:** Memory template ready.

---

### Step 4: Create Quick-Start Hook Template (3 min)

```bash
cat > hooks/detect-secrets-template.sh <<'EOF'
#!/bin/bash
# PreToolUse hook: Detect secrets in code
# Customize patterns for your banking IT requirements

SECRETS_PATTERNS=(
    "password\s*=\s*['\"][^'\"]*['\"]"
    "api[_-]?key\s*=\s*['\"][^'\"]*['\"]"
    "secret\s*=\s*['\"][^'\"]*['\"]"
    "aws_access_key_id"
    "sk-[a-zA-Z0-9]{20,}"
    "[0-9]{13,19}"  # Credit card numbers
    "[0-9]{3}-[0-9]{2}-[0-9]{4}"  # SSN format
)

for pattern in "${SECRETS_PATTERNS[@]}"; do
    if grep -rE "$pattern" . 2>/dev/null | grep -v -E "(.claude/|test_|.git/)"; then
        echo "❌ SECURITY VIOLATION: Potential secret detected!"
        echo "Pattern matched: $pattern"
        echo "BLOCKED by PreToolUse hook"
        exit 1
    fi
done

exit 0
EOF

chmod +x hooks/detect-secrets-template.sh
echo "✅ Secrets detection hook template created"
```

**✅ Checkpoint 4:** Hook template ready.

---

### Step 5: Use Template in New Project (2 min)

```bash
# Create new project from templates
mkdir -p ~/my-new-pipeline && cd ~/my-new-pipeline
mkdir -p .claude/hooks

# Copy templates
cp ~/team-claude-templates/settings/banking-it-standard.json .claude/settings.json
cp ~/team-claude-templates/memory/CLAUDE-banking-template.md .claude/CLAUDE.md
cp ~/team-claude-templates/hooks/detect-secrets-template.sh .claude/hooks/detect-secrets.sh

# Customize CLAUDE.md
sed -i '' 's/{{PROJECT_NAME}}/Payment Processing Pipeline/' .claude/CLAUDE.md
sed -i '' 's/{{TEAM_NAME}}/Data Engineering Team/' .claude/CLAUDE.md
sed -i '' 's/{{DOMAIN}}/Payment Processing/' .claude/CLAUDE.md

# Test it
claude
> Hello, what project am I working on?
# Claude should reference "Payment Processing Pipeline"!

Ctrl+D
```

**✅ Checkpoint 5:** Template successfully deployed to new project!

---

### ✅ Success Criteria

Template library successfully created when:
- ✅ Settings template includes banking IT permissions
- ✅ Memory template documents standards
- ✅ Hook templates enforce security
- ✅ Templates can be copied to new projects
- ✅ Placeholders ({{NAME}}) easily customizable
- ✅ Team can share templates via git

**Result:** Your team now has standardized, reusable Claude Code templates!

---

## Table of Contents
1. [Settings Templates](#settings-templates)
2. [Memory Templates](#memory-templates)
3. [Slash Command Templates](#slash-command-templates)
4. [Hook Script Templates](#hook-script-templates)
5. [Agent Configuration Templates](#agent-configuration-templates)
6. [Quick Start Templates](#quick-start-templates)

---

## Settings Templates

### Template 1: Standard Development

`.claude/settings.json`:
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
          "command": "python ./.claude/hooks/security-check.py",
          "description": "Security validation before code changes"
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
  "$schema": "https://claude.ai/schemas/settings.json",
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch", "WebSearch"]
  },
  "defaultModel": "sonnet",
  "env": {
    "ENVIRONMENT": "production",
    "READ_ONLY": "true"
  },
  "hooks": {
    "PreToolUse": [{
      "matcher": ".*",
      "hooks": [{
        "type": "command",
        "command": "python ./.claude/hooks/audit-log.py",
        "description": "Audit all production access"
      }]
    }]
  }
}
```

### Template 3: Security Audit

`.claude/settings.json`:
```json
{
  "$schema": "https://claude.ai/schemas/settings.json",
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": "*"
  },
  "defaultModel": "sonnet",
  "outputStyle": "security-focused",
  "agents": {
    "security-auditor": {
      "description": "Security vulnerability scanner for data pipelines",
      "prompt": "You are a security expert specializing in PySpark data pipelines. Identify vulnerabilities: SQL injection, hardcoded secrets, PII exposure, insecure authentication.",
      "tools": ["Read", "Grep", "Glob"]
    },
    "compliance-checker": {
      "description": "PCI-DSS, SOX, GDPR compliance auditor",
      "prompt": "You are a banking compliance expert. Check for: PCI-DSS violations (CVV storage, card data exposure), SOX requirements (audit trails), GDPR violations (PII handling).",
      "tools": ["Read", "Grep", "Glob"]
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
{{Brief description of the data pipeline project - what it does, what business problems it solves}}

## Technology Stack
- **Language**: Python 3.10+
- **Framework**: PySpark 3.5+
- **Storage**: Delta Lake 3.0+ / Parquet
- **Database**: Snowflake / PostgreSQL / Redshift
- **Testing**: pytest, pytest-spark
- **Code Quality**: Ruff, Black, mypy
- **Orchestration**: Databricks Workflows / Apache Airflow

## Coding Standards

### Python Style
- Follow PEP 8 style guide (enforced by Ruff)
- Format with Black (line length: 100)
- Use type hints for all function signatures
- 4 spaces for indentation
- Docstrings required for all public functions (Google style)

### PySpark Best Practices
- Use DataFrame API (not RDD API)
- Avoid `collect()` on large datasets (causes OOM)
- Use `broadcast()` for small lookup tables (<10MB)
- Partition data appropriately (target: 128MB per partition)
- Cache intermediate DataFrames when reused 2+ times
- Prefer built-in functions over UDFs for performance
- Define explicit schemas with StructType

**Example:**
```python
from pyspark.sql.types import StructType, StructField, StringType, DecimalType

# Good: Explicit schema
schema = StructType([
    StructField("transaction_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False)
])
df = spark.read.schema(schema).parquet(path)

# Bad: Schema inference (slow and error-prone)
df = spark.read.parquet(path)
```

## Security Requirements
- All data access requires authentication (service principals, not user credentials)
- Use parameterized queries for all SQL operations (prevent SQL injection)
- Never log PII: account numbers, SSNs, customer names, email addresses
- Encrypt data at rest (AES-256) and in transit (TLS 1.2+)
- Validate all input data schemas with StructType
- Mask sensitive data in non-production environments

## Testing Standards
- Minimum test coverage: 80% (enforced in CI/CD)
- Write tests before implementation (TDD preferred)
- Mock external data sources (S3, Snowflake, Kafka)
- Use small sample datasets for unit tests (<1000 records)
- Integration tests on staging cluster with realistic data volumes
- Use `pyspark.testing.utils.assertDataFrameEqual` for DataFrame comparisons

## Git Workflow
- Branch naming: `feature/`, `fix/`, `refactor/`, `perf/`, `test/`
- Commit format: Conventional Commits
- Require 2 PR approvals for production code changes
- All git operations performed manually (no automation)

## Compliance
- **PCI-DSS**: Tokenize card data, never store CVV/CVV2
- **SOX**: Maintain audit trails for financial transactions
- **GDPR**: Support right to deletion, enforce data retention policies
```

### Template 2: Banking Data Engineering Project Memory

`.claude/CLAUDE.md`:
```markdown
# {{BANKING_PROJECT_NAME}} - Data Engineering Pipelines

## Project Overview
Real-time and batch data processing pipelines for banking transactions, customer accounts, and payment processing with PCI-DSS and SOX compliance.

## Compliance Requirements

### PCI-DSS (Payment Card Industry Data Security Standard)

**NEVER:**
- ❌ Log credit card numbers (full or partial) anywhere
- ❌ Store CVV/CVV2/CVC codes (must NEVER be persisted)
- ❌ Store PIN blocks in logs or temporary tables
- ❌ Include full card numbers in error messages or DataFrames shown in notebooks

**ALWAYS:**
- ✅ Tokenize card numbers before storage (use tokenization service)
- ✅ Encrypt cardholder data (CHD) with AES-256
- ✅ Mask card numbers in logs: show only last 4 digits (`****1234`)
- ✅ Audit all access to payment data (log to immutable audit table)
- ✅ Use network segmentation (private endpoints only)

**Example:**
```python
# Good: Masked logging
logger.info(f"Processing payment for card: ****{card_token[-4:]}")

# Bad: Exposing card data
logger.info(f"Processing card: {card_number}")  # NEVER DO THIS!
```

### SOX Compliance (Sarbanes-Oxley)
- All financial transactions must be auditable (immutable audit log)
- Maintain complete data lineage (document source → transformations → destination)
- Change management required for production pipelines (PR approvals)
- Separation of duties enforced (dev cannot deploy to prod)
- Data integrity checks mandatory (checksums, row counts)

### GDPR (General Data Protection Regulation)
- Personal data encrypted at rest and in transit
- Right to deletion implemented (data purge pipelines with soft deletes)
- Data retention: 7 years for financial records, 90 days for marketing data
- Data residency requirements enforced (EU data stays in EU)
- Consent tracking for marketing data (consent_flag in customer table)

## Data Security Standards

### Authentication & Authorization
- **Service Principals**: Use managed identities (not user credentials)
- **Role-Based Access**: Least privilege principle enforced
- **Encryption**: AES-256 for data at rest, TLS 1.2+ for in transit
- **Network Security**: Private endpoints only, no public access to data lakes

### PySpark Security Patterns

```python
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StructField, StringType, DecimalType
import logging

logger = logging.getLogger(__name__)

# GOOD: Masked logging for PII
def log_transaction_safely(account_id: str, amount: Decimal) -> None:
    """Log transaction with PII masking."""
    masked_account = f"****{account_id[-4:]}"
    logger.info(f"Processing transaction: account={masked_account}, amount=${amount}")

# BAD: Exposing PII in logs
def log_transaction_unsafe(account_id: str, amount: Decimal) -> None:
    logger.info(f"Transaction: {account_id}, {amount}")  # NEVER!

# GOOD: Schema validation with StructType
transaction_schema = StructType([
    StructField("transaction_id", StringType(), nullable=False),
    StructField("account_id", StringType(), nullable=False),
    StructField("amount", DecimalType(18, 2), nullable=False),  # Use Decimal, not Float!
    StructField("currency", StringType(), nullable=False)
])

df = spark.read.schema(transaction_schema).parquet("s3://transactions/")

# GOOD: Data quality checks
invalid_amounts = df.filter(F.col("amount") < 0).count()
assert invalid_amounts == 0, f"Found {invalid_amounts} negative amounts - data quality violation"

# GOOD: Masking PII in DataFrames for non-prod
if environment != "production":
    df = df.withColumn(
        "account_id_masked",
        F.concat(F.lit("****"), F.substring(F.col("account_id"), -4, 4))
    ).drop("account_id")
```

## Code Review Requirements

**Security Review Required For:**
- Authentication/authorization code
- Payment data processing pipelines
- SQL queries accessing sensitive tables (accounts, transactions, customers)
- External API integrations (payment gateways, fraud detection)
- Data masking/tokenization logic

**Automated Checks:**
- Security scan must pass (no high/critical vulnerabilities)
- Dependency vulnerability scan (`pip-audit`)
- Secrets detection (`/detect-secrets`)
- PCI compliance check (`/compliance-check`)

**Manual Reviews:**
- Data lineage documented (source → destination)
- Performance tested on staging cluster
- Monitoring/alerting configured

## Testing Standards

### Coverage Requirements
- Unit test coverage: ≥80% (enforced in CI/CD)
- Integration tests for end-to-end pipelines
- Data quality tests (nulls, ranges, referential integrity)
- Performance tests for large datasets (1M+ records)

### Test Data
- **Never use real card numbers** (use test data: `4111111111111111` for Visa)
- **Never use real customer PII** (generate synthetic data with Faker)
- Mock external services (payment gateways, fraud detection APIs)
- Use small datasets for unit tests (<1000 records)

### Example Unit Test
```python
import pytest
from pyspark.sql import SparkSession
from pyspark.testing.utils import assertDataFrameEqual
from pipelines.validators import validate_transactions

@pytest.fixture(scope="session")
def spark():
    return SparkSession.builder.master("local[2]").getOrCreate()

def test_validate_transactions_filters_invalid_amounts(spark):
    """Test that negative amounts are filtered out."""
    # Arrange
    input_data = [
        ("TXN001", "ACC123", 100.00),
        ("TXN002", "ACC456", -50.00),  # Invalid
        ("TXN003", "ACC789", 200.00)
    ]
    input_df = spark.createDataFrame(input_data, ["txn_id", "account_id", "amount"])

    # Act
    result_df = validate_transactions(input_df)

    # Assert
    assert result_df.count() == 2
    assert result_df.filter(F.col("amount") < 0).count() == 0
```

## Pipeline Standards

### Idempotency
- All pipelines must support reruns without duplicating data
- Use upsert (merge) operations for Delta Lake
- Include idempotency keys in all transactions

### Incremental Processing
- Prefer incremental loads over full refreshes
- Use watermarking for streaming pipelines
- Implement checkpointing for Structured Streaming

### Partitioning & Performance
- Partition by date for time-series data
- Z-ORDER by commonly filtered columns (account_id, customer_id)
- Target 128MB per partition
- Use broadcast joins for small reference tables (<10MB)

### Data Quality
- Schema validation at ingestion (StructType enforcement)
- Business rule validation (amount ranges, currency codes)
- Referential integrity checks (foreign keys exist)
- Completeness checks (required fields non-null)

### Monitoring & Alerting
- SLA: 99.9% uptime for critical pipelines
- Alert on validation failure rate >5%
- Alert on pipeline duration >2x baseline
- Dashboard for data quality metrics (completeness, accuracy, timeliness)

## Incident Response

When production issues occur:
1. **Investigate** in read-only mode (`claude --permission-mode plan`)
2. **Develop fix** in dev environment (never directly in prod)
3. **Test thoroughly** on staging cluster
4. **Deploy** via standard release process (no hotfixes to prod)
5. **Document** in postmortem (root cause, remediation, prevention)
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
- Are data sources properly authenticated with service principals?
- Is row-level security implemented where required?
- Are permissions following least privilege principle?
- Secure credential management (Key Vault, environment variables)?

## Data Protection
- PII properly masked/encrypted in non-production environments?
- Card data tokenized (no raw card numbers)?
- Secrets managed in key vault (no hardcoded credentials)?
- Data at rest encrypted (AES-256)?
- Data in transit encrypted (TLS 1.2+)?

## Input Validation
- Schema validation enforced with StructType?
- Data quality checks present (nulls, ranges)?
- SQL injection prevention (parameterized queries only)?
- Column-level security enforced (masking in queries)?

## Logging & Monitoring
- Security events logged to audit table?
- PII excluded from logs (masked account numbers)?
- Sensitive data properly masked in error messages?
- Audit trails for data access maintained?

## Compliance
- **PCI-DSS**: No CVV storage, card data tokenized?
- **SOX**: Audit trail maintained for financial transactions?
- **GDPR**: Right-to-deletion supported, data retention policies enforced?

Provide file:line references and specific remediation steps.
```

### Template 2: Generate Tests

`.claude/commands/generate-tests.md`:
```markdown
---
description: Generate comprehensive pytest tests - Usage: /generate-tests <file>
---

Generate unit tests for: $1

Include:
- **Happy path tests**: Valid inputs, expected outputs
- **Edge cases**: Nulls, empty DataFrames, schema mismatches, boundary values
- **Error conditions**: Missing files, invalid data, network failures
- **Data quality validations**: Duplicate detection, referential integrity
- **Mock external dependencies**: S3, Snowflake, Kafka

Use pytest framework with `pyspark.testing.utils.assertDataFrameEqual`.
Follow AAA pattern (Arrange, Act, Assert).

**Example structure:**
```python
import pytest
from pyspark.sql import SparkSession
from pyspark.testing.utils import assertDataFrameEqual
from pyspark.sql import functions as F

@pytest.fixture(scope="session")
def spark():
    """Create Spark session for testing."""
    return SparkSession.builder.master("local[2]").getOrCreate()

@pytest.fixture
def sample_transactions(spark):
    """Sample transaction data."""
    data = [
        ("TXN001", "ACC123", 100.00, "USD"),
        ("TXN002", "ACC456", 200.00, "USD")
    ]
    schema = ["txn_id", "account_id", "amount", "currency"]
    return spark.createDataFrame(data, schema)

def test_transform_happy_path(spark, sample_transactions):
    """Test transformation with valid data."""
    # Arrange
    input_df = sample_transactions

    # Act
    result_df = transform_function(input_df)

    # Assert
    expected_data = [("TXN001", "ACC123", 100.00, "USD", True)]
    expected_df = spark.createDataFrame(expected_data, ["txn_id", "account_id", "amount", "currency", "processed"])
    assertDataFrameEqual(result_df, expected_df)

def test_transform_handles_nulls(spark):
    """Test handling of null values."""
    # Arrange
    data = [("TXN001", None, 100.00, "USD")]  # null account_id
    input_df = spark.createDataFrame(data, ["txn_id", "account_id", "amount", "currency"])

    # Act
    result_df = transform_function(input_df)

    # Assert
    assert result_df.filter(F.col("account_id").isNull()).count() == 0
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
- **Purpose**: What business problem does this solve?
- **Data sources**: Input tables/files/APIs with schemas
- **Data destinations**: Output tables/files with schemas
- **Schedule/trigger**: Batch (daily/hourly), streaming, event-driven

## Data Lineage
- **Input datasets**: Names, schemas (StructType), sample data
- **Transformation logic**: Step-by-step with PySpark code examples
- **Output datasets**: Names, schemas, partitioning strategy
- **Dependencies**: Upstream pipelines this depends on, downstream consumers

## Transformations
- **Data quality checks**: Nulls, ranges, duplicates, referential integrity
- **Business rules**: Validation logic (amounts, currencies, statuses)
- **Aggregations and joins**: Window functions, group by, join strategies
- **Partitioning strategy**: Partition columns, target size per partition

## Performance
- **SLA requirements**: Runtime target, throughput target
- **Resource allocation**: Cluster size, executor memory/cores
- **Optimization**: Broadcast joins, caching, Z-ORDER

## Monitoring
- **Data quality metrics**: Completeness, accuracy, timeliness
- **Performance metrics**: Runtime, throughput, latency
- **Alert conditions**: Failure rate >5%, runtime >2x baseline

## Security & Compliance
- **PII handling**: Masking strategy, encryption
- **Access controls**: Service principals, permissions
- **Compliance**: PCI-DSS (tokenization), SOX (audit trail), GDPR (retention)

## Error Handling
- **Retry logic**: Exponential backoff, max retries
- **Dead letter queue**: Invalid records written to error table
- **Alerting**: PagerDuty/email on failures

Format as Markdown with:
- Data flow diagram (ASCII art)
- Code examples for key transformations
- Sample queries for business users
```

---

## Hook Script Templates

### Template 1: Audit Logging

`.claude/hooks/audit-log.py`:
```python
#!/usr/bin/env python3
"""Audit logging for compliance - logs all tool usage."""

import os
import sys
from datetime import datetime, timezone
from pathlib import Path
import json

def main():
    """Log tool usage to local audit file and optionally to central system."""
    timestamp = datetime.now(timezone.utc).isoformat()
    user = os.environ.get('USER', os.getlogin())
    tool_name = os.environ.get('TOOL_NAME', sys.argv[1] if len(sys.argv) > 1 else 'unknown')
    file_path = os.environ.get('FILE_PATH', sys.argv[2] if len(sys.argv) > 2 else '')
    project = Path.cwd().name

    # Create log directory
    log_file = Path.home() / '.claude' / 'audit.log'
    log_file.parent.mkdir(parents=True, exist_ok=True)

    # Log entry
    log_entry = {
        "timestamp": timestamp,
        "user": user,
        "project": project,
        "tool": tool_name,
        "file": file_path
    }

    # Append to local log
    with open(log_file, 'a') as f:
        f.write(json.dumps(log_entry) + '\n')

    # Optional: Send to central logging system
    # import requests
    # requests.post('https://audit.company.com/api/logs', json=log_entry)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### Template 2: Secrets Detection

`.claude/hooks/detect-secrets.py`:
```python
#!/usr/bin/env python3
"""Detect hardcoded secrets in Python/PySpark code."""

import os
import sys
import re
from pathlib import Path

def main():
    """Check for common secret patterns in code."""
    file_path = os.environ.get('FILE_PATH', sys.argv[1] if len(sys.argv) > 1 else '')

    if not file_path or not os.path.exists(file_path):
        sys.exit(0)

    # Only check Python files
    if not file_path.endswith('.py'):
        sys.exit(0)

    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        content = f.read()

    # Check for common secret patterns
    secret_patterns = [
        (r'(password|secret|api_key|token|access_key)\s*=\s*["\'][^"\']{8,}["\']',
         "Potential hardcoded secret"),
        (r'AKIA[0-9A-Z]{16}',
         "AWS Access Key"),
        (r'AccountKey=[A-Za-z0-9+/=]{88}',
         "Azure Storage Account Key"),
        (r'(jdbc|mysql|postgresql|snowflake)://[^:]+:[^@]+@',
         "Database connection string with credentials"),
        (r'sk-[A-Za-z0-9]{32,}',
         "OpenAI API key"),
        (r'-----BEGIN PRIVATE KEY-----',
         "Private key"),
    ]

    for pattern, description in secret_patterns:
        if re.search(pattern, content, re.IGNORECASE):
            print(f"ERROR: {description} detected in {file_path}")
            print("Use environment variables or key vault instead.")
            sys.exit(1)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### Template 3: PCI Compliance Check

`.claude/hooks/pci-compliance.py`:
```python
#!/usr/bin/env python3
"""Check for PCI-DSS violations in PySpark code."""

import os
import sys
import re

def main():
    """Check for PCI-DSS compliance violations."""
    file_path = os.environ.get('FILE_PATH', sys.argv[1] if len(sys.argv) > 1 else '')

    if not file_path or not os.path.exists(file_path):
        sys.exit(0)

    # Only check Python files
    if not file_path.endswith('.py'):
        sys.exit(0)

    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        content = f.read()

    violations = []

    # Check for credit card numbers in logs
    cc_in_log = re.compile(r'(log|print|logger).*\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b', re.IGNORECASE)
    if cc_in_log.search(content):
        violations.append("CRITICAL: Credit card number in logging detected!")

    # Check for CVV storage
    cvv_storage = re.compile(r'(cvv|cvc|card.*verification).*(store|save|insert|update|write)', re.IGNORECASE)
    if cvv_storage.search(content):
        violations.append("CRITICAL: Potential CVV storage detected! CVV must NEVER be stored (PCI-DSS 3.2)")

    # Check for unmasked card display
    card_display = re.compile(r'\.(show|display|print).*card', re.IGNORECASE)
    if card_display.search(content):
        violations.append("WARNING: Displaying card data - ensure masking is applied")

    # Check for PySpark DataFrame operations with sensitive data
    if 'pyspark' in content.lower():
        # Check for .show() on card columns
        sensitive_show = re.compile(r'\\.select\\([^)]*(?:card|cvv|pan)[^)]*\\)\\.show\\(', re.IGNORECASE)
        if sensitive_show.search(content):
            violations.append("WARNING: Displaying sensitive card data with .show() - use .show(truncate=True) or mask data")

    if violations:
        print(f"PCI-DSS violations found in {file_path}:")
        for violation in violations:
            print(f"  - {violation}")
        sys.exit(1)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

---

## Agent Configuration Templates

### Banking Data Engineering Agent Suite

`.claude/settings.json`:
```json
{
  "agents": {
    "compliance-checker": {
      "description": "PCI-DSS, SOX, GDPR compliance auditor for data pipelines",
      "prompt": "You are a banking compliance expert for data engineering. Check PySpark pipelines and Python code for:\n\n**PCI-DSS:**\n- CVV storage (must NEVER be stored)\n- Unencrypted card data\n- Card numbers in logs\n- Missing tokenization\n\n**SOX:**\n- Missing audit trails\n- Incomplete data lineage\n- Lack of change management\n\n**GDPR:**\n- Unencrypted PII\n- Missing consent tracking\n- No data retention policy\n\nProvide file:line references, severity (Critical/High/Medium/Low), and remediation steps.",
      "tools": ["Read", "Grep", "Glob"]
    },

    "data-quality-auditor": {
      "description": "Data quality and pipeline reliability specialist",
      "prompt": "You are a data quality expert for PySpark pipelines. Identify:\n- Missing null checks\n- Schema validation gaps\n- Referential integrity issues\n- Missing data quality tests\n- Inadequate error handling\n- Performance bottlenecks (collect(), cartesian joins)\n\nProvide specific recommendations with PySpark code examples.",
      "tools": ["Read", "Grep", "Glob"]
    },

    "security-reviewer": {
      "description": "Data security auditor for PySpark pipelines",
      "prompt": "You are a data security expert. Identify:\n- Unmasked PII in logs or DataFrames\n- Hardcoded secrets (passwords, API keys)\n- SQL injection risks (string concatenation in queries)\n- Insecure data access patterns\n- Missing encryption (at rest, in transit)\n- Over-permissioned service principals\n\nProvide file:line references and secure alternatives.",
      "tools": ["Read", "Grep", "Glob"]
    },

    "pyspark-optimizer": {
      "description": "PySpark performance optimization specialist",
      "prompt": "You are a PySpark performance expert. Identify:\n- Unnecessary collect() calls (causes OOM)\n- Missing broadcast hints for small tables\n- Poor partitioning (too many/few partitions)\n- Redundant shuffles\n- Missing caching for reused DataFrames\n- Inefficient joins (sort-merge vs broadcast)\n- UDFs that could be built-in functions\n\nSuggest optimizations with before/after code examples.",
      "tools": ["Read", "Grep", "Glob"]
    },

    "test-generator": {
      "description": "Pytest test creation specialist for PySpark",
      "prompt": "You generate comprehensive pytest tests for PySpark pipelines. Include:\n- Happy paths with valid data\n- Edge cases (nulls, empty DataFrames, schema mismatches)\n- Data quality checks (duplicates, referential integrity)\n- Error conditions (missing files, network failures)\n\nUse pyspark.testing.utils.assertDataFrameEqual and fixtures. Follow AAA pattern.",
      "tools": ["Read", "Write", "Grep"]
    },

    "doc-generator": {
      "description": "Data pipeline documentation specialist",
      "prompt": "You generate clear pipeline documentation including:\n- Data lineage (source → transformations → destination)\n- Schemas (StructType definitions)\n- Business logic and validation rules\n- Performance characteristics (SLA, throughput)\n- Monitoring and alerting\n- Security and compliance notes\n\nUse Markdown with ASCII diagrams and code examples.",
      "tools": ["Read", "Write", "Grep"]
    }
  }
}
```

---

## Quick Start Templates

### New Data Pipeline Project Setup Script

`scripts/setup-claude-code.sh`:
```bash
#!/bin/bash
# setup-claude-code.sh - Initialize Claude Code for PySpark data pipeline project

set -e

echo "Initializing Claude Code for data engineering project..."

# Create directory structure
mkdir -p .claude/{commands,hooks,output-styles}
mkdir -p .claude/commands/{review,generate,quality}
mkdir -p pipelines/{etl,transformers,validators}
mkdir -p tests/{unit,integration}

# Create settings.json
cat > .claude/settings.json << 'EOF'
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
    "PYSPARK_PYTHON": "python3"
  }
}
EOF

# Create CLAUDE.md
cat > .claude/CLAUDE.md << 'EOF'
# Data Pipeline Project

## Technology Stack
- Python 3.10+
- PySpark 3.5+
- Delta Lake 3.0+
- pytest, Ruff, Black, mypy

## Coding Standards
- Style: PEP 8 (enforced by Ruff)
- Formatting: Black (line length: 100)
- Type hints: Required for all functions
- Testing: pytest with 80% coverage

## PySpark Best Practices
- Use DataFrame API (not RDD)
- Avoid collect() on large datasets
- Broadcast small lookup tables
- Partition appropriately (128MB target)

## Security
- Mask PII in logs
- Use environment variables for secrets
- Validate schemas with StructType
- Encrypt at rest and in transit

## Compliance
- PCI-DSS: Tokenize card data, no CVV storage
- SOX: Maintain audit trails
- GDPR: Support data deletion, retention policies
EOF

# Create pyproject.toml
cat > pyproject.toml << 'EOF'
[project]
name = "data-pipelines"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = [
    "pyspark>=3.5.0",
    "delta-spark>=3.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "pytest-spark>=0.6.0",
    "ruff>=0.1.0",
    "black>=23.0.0",
    "mypy>=1.5.0",
]

[tool.black]
line-length = 100
target-version = ['py310', 'py311']

[tool.ruff]
line-length = 100
target-version = "py310"

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "C4", "UP", "S"]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=pipelines --cov-report=html --cov-fail-under=80"
EOF

# Create .gitignore
cat > .gitignore << 'EOF'
# Claude Code
.claude/settings.local.json
.claude/.sessions/

# Python
__pycache__/
*.pyc
.venv/
venv/

# PySpark
derby.log
metastore_db/
spark-warehouse/

# Testing
.pytest_cache/
.coverage
htmlcov/
.mypy_cache/
.ruff_cache/

# Secrets
.env
*.key
credentials.json
EOF

# Create conftest.py for pytest
cat > tests/conftest.py << 'EOF'
import pytest
from pyspark.sql import SparkSession


@pytest.fixture(scope="session")
def spark():
    """Create Spark session for testing."""
    return (
        SparkSession.builder
        .master("local[2]")
        .appName("pytest-pyspark")
        .config("spark.sql.shuffle.partitions", "2")
        .getOrCreate()
    )
EOF

echo "✅ Claude Code initialized successfully!"
echo ""
echo "Next steps:"
echo "1. Edit .claude/CLAUDE.md with your project standards"
echo "2. Install dependencies: pip install -e '.[dev]'"
echo "3. Start Claude Code: claude"
echo "4. Run tests: pytest"
echo "5. Check code quality: ruff check . && black --check . && mypy pipelines/"
```

Make executable:
```bash
chmod +x scripts/setup-claude-code.sh
```

---

## Summary

In this subsection, you learned:

### Templates Provided
- ✅ **Settings templates**: Development, production read-only, security audit
- ✅ **Memory templates**: Basic project standards, banking compliance requirements
- ✅ **Slash command templates**: Security review, test generation, pipeline documentation
- ✅ **Hook script templates**: Audit logging, secrets detection, PCI compliance
- ✅ **Agent templates**: Complete suite for banking data engineering
- ✅ **Quick start script**: Automated project setup

### How to Use Templates
1. Copy templates to your project
2. Replace `{{placeholders}}` with your values
3. Customize for your team's needs
4. Commit to git for team sharing
5. Update as requirements evolve

---

## Next Steps

👉 **Phase 1.3 Complete!** Continue to **[Phase 1.4: Prompt Engineering for Data Engineering](../04-prompt-engineering-data-engineering.md)**

**Quick Actions:**
1. Run `scripts/setup-claude-code.sh` in a new project
2. Customize `.claude/CLAUDE.md` for your team
3. Add custom slash commands for your workflows
4. Configure agents for your specific needs

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Configuration details
- [Phase 1.3.6: Slash Commands](./03-06-slash-commands.md) - Custom commands
- [Phase 1.3.7: Agents & Sub-agents](./03-07-agents-subagents.md) - Agent configuration
- [Phase 1.3.8: Hooks & Automation](./03-08-hooks-automation.md) - Hook scripts
- [Phase 1.3.11: Standards & Best Practices](./03-11-standards-best-practices.md) - Team standards

---

**Last Updated:** 2025-10-24
**Version:** 1.0
