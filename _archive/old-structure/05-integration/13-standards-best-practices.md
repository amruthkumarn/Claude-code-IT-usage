# Section 13: Standards & Best Practices

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
├── compliance-check.md      # Clear, descriptive
├── security-review.md        # Hyphenated
├── data-quality-check.md    # Hyphenated, descriptive
└── generate-tests.md        # Verb-noun format
```

**Memory Files:**
```
CLAUDE.md                    # Main project memory
.claude/CLAUDE.md           # Alternative location
~/.claude/CLAUDE.md         # User memory
```

**Configuration Files:**
```
.claude/settings.json        # Team-shared
.claude/settings.local.json  # Personal (gitignored)
```

### File Organization

```
your-pyspark-project/
├── .claude/
│   ├── settings.json              # Team configuration
│   ├── settings.local.json        # Personal (in .gitignore)
│   ├── CLAUDE.md                  # Project memory
│   ├── commands/                  # Custom slash commands
│   │   ├── README.md              # Command documentation
│   │   ├── review/                # Organized by category
│   │   │   ├── security.md
│   │   │   └── compliance.md
│   │   └── generate/
│   │       ├── tests.md
│   │       └── docs.md
│   ├── hooks/                     # Hook scripts
│   │   ├── README.md
│   │   ├── audit-log.sh
│   │   └── security-check.sh
│   └── output-styles/             # Custom output styles
│       └── detailed.md
├── .gitignore                     # Include .claude/settings.local.json
└── README.md                      # Document Claude Code usage
```

---

## Project Setup

### Initial Setup Checklist

**1. Create .claude directory:**
```bash
mkdir -p .claude/{commands,hooks,output-styles}
```

**2. Create settings.json:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": []
  },
  "defaultModel": "sonnet",
  "hooks": {}
}
```

**3. Create CLAUDE.md:**
```markdown
# Project Name

## Coding Standards
- Use Python 3.8+ with type hints
- Follow PEP 8 style guide (enforced by Ruff)
- Format code with Black
- Use PySpark for distributed data processing
- Follow functional programming patterns

## Security Requirements
- No credentials in code
- Use environment variables or secret managers
- Parameterized SQL queries only
- Input validation for all external data

## Testing Standards
- Unit tests with pytest
- Integration tests for PySpark jobs
- Test coverage ≥ 80%
- Data quality tests for all pipelines
```

**4. Update .gitignore:**
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
```

**5. Create README section:**
```markdown
## Claude Code Setup

This project uses Claude Code for AI-assisted development.

### Getting Started
1. Install: `pip install claude-code` or use your package manager
2. Login: `claude /login`
3. Start: `claude`

### Custom Commands
- `/compliance-check` - Check PCI-DSS/SOX compliance
- `/security-review` - Security audit
- `/data-quality-check` - Validate data pipeline quality
- `/generate-tests` - Generate pytest test cases

See `.claude/commands/README.md` for full list.
```

---

## Team Collaboration

### Shared Configuration

**What to commit:**
- `.claude/settings.json` - Team settings
- `.claude/CLAUDE.md` - Project standards
- `.claude/commands/` - Custom commands
- `.claude/hooks/` - Hook scripts
- `.claude/output-styles/` - Custom styles
- `pyproject.toml` or `setup.py` - Project dependencies
- `requirements.txt` or `poetry.lock` - Locked dependencies

**What NOT to commit:**
- `.claude/settings.local.json` - Personal settings
- `.claude/.sessions/` - Session history
- `venv/` or `.venv/` - Virtual environments
- `__pycache__/` - Python bytecode
- `.env` - Environment variables

### Code Review with Claude

**Reviewer checklist:**
```markdown
# Code Review Checklist

Before approving PR:
- [ ] Run `/security-review` on changed files
- [ ] Run `/compliance-check` for regulatory code
- [ ] Verify test coverage with `/test-coverage`
- [ ] Run `/data-quality-check` for pipeline changes
- [ ] Check for secrets with `/detect-secrets`
- [ ] Verify Ruff linting passes
- [ ] Verify Black formatting applied
- [ ] Verify mypy type checking passes
- [ ] Review Claude Code audit log for this session
```

### Onboarding New Team Members

**Day 1 - Setup:**
```markdown
1. Install Claude Code
2. Clone project
3. Create virtual environment: `python -m venv venv`
4. Activate: `source venv/bin/activate` (Unix) or `venv\Scripts\activate` (Windows)
5. Install dependencies: `pip install -r requirements.txt`
6. Run `claude` in project directory
7. Try `/help` to see custom commands
8. Read `.claude/CLAUDE.md` for standards
```

**Day 2 - Practice:**
```markdown
1. Use Plan Mode to explore: `claude --permission-mode plan`
2. Ask questions about the codebase and PySpark pipelines
3. Try fixing a small bug with Claude's help
4. Review changes carefully before approving
5. Run tests locally: `pytest`
6. Check linting: `ruff check .`
7. Format code: `black .`
```

---

## Security Best Practices

### 1. Principle of Least Privilege

**Development:**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  }
}
```

**Production (Read-Only):**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash", "WebFetch"]
  }
}
```

### 2. Secrets Management

**Never:**
```python
# config.py - WRONG!
API_KEY = "sk-1234567890"  # NEVER hardcode secrets!
DB_PASSWORD = "mypassword123"  # NEVER!
AWS_SECRET_KEY = "abcd1234"  # NEVER!
```

**Always:**
```python
# config.py - CORRECT
import os
from typing import Optional

def get_secret(key: str, default: Optional[str] = None) -> str:
    """Get secret from environment variable."""
    value = os.getenv(key, default)
    if value is None:
        raise ValueError(f"Secret {key} not found in environment")
    return value

API_KEY = get_secret("API_KEY")
DB_PASSWORD = get_secret("DB_PASSWORD")
AWS_SECRET_KEY = get_secret("AWS_SECRET_KEY")
```

**Settings configuration:**
```json
{
  "env": {
    "API_KEY": "${API_KEY}",
    "DB_PASSWORD": "${DB_PASSWORD}"
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
        "command": "./.claude/hooks/audit-log.sh"
      }]
    }]
  }
}
```

### 4. Regular Security Reviews

```bash
# Monthly security audit
claude --permission-mode plan

> @security-auditor perform comprehensive security review

> Check for:
> - SQL injection vulnerabilities
> - Hardcoded secrets and credentials
> - Authentication and authorization issues
> - Sensitive data exposure in logs
> - Insecure data serialization
> - Unvalidated external inputs
```

---

## Performance Optimization

### 1. Use Available Model

```bash
# Sonnet is currently the only available model in AWS Bedrock
# Suitable for all development tasks: quick queries, development, complex analysis
claude

# Or explicitly specify
claude --model sonnet

# For read-only analysis
claude --permission-mode plan
> Analyze the PySpark pipeline architecture and suggest optimizations
```

**Note:** Only Sonnet is currently available in AWS Bedrock. It provides excellent performance for all development tasks.

### 2. Manage Context

**Keep sessions focused:**
```bash
# Don't:
> Tell me about pipeline1.py
> Now pipeline2.py
> Now pipeline3.py
# ... (context bloat)

# Do:
> Explain the transaction processing pipeline (files: pipelines/transactions/*)
```

**Start fresh when needed:**
```bash
# If Claude "forgets" things
Ctrl+D  # Exit
claude  # New session, clean context
```

### 3. Use Agents for Subtasks

```bash
# Instead of one long session:
> @security-auditor review security
> @test-generator create pytest tests
> @doc-writer generate documentation
> @data-validator check data quality

# Each agent has clean context
```

---

## Banking-Specific Standards

### Code Review Standards

```markdown
# Banking Code Review Checklist

## Security
- [ ] No hardcoded credentials
- [ ] SQL queries use parameterized statements
- [ ] Authentication required on endpoints
- [ ] Input validation implemented
- [ ] Error messages don't leak sensitive info
- [ ] PII/PCI data properly encrypted

## Compliance
- [ ] PCI-DSS: No card data in logs or temporary storage
- [ ] SOX: Audit trail for financial transactions
- [ ] GDPR: Personal data properly handled with consent
- [ ] No CVV storage anywhere
- [ ] Data retention policies enforced

## Data Quality
- [ ] Schema validation for all inputs
- [ ] Null handling implemented
- [ ] Data type validation
- [ ] Duplicate detection logic
- [ ] Data lineage documented

## Code Quality
- [ ] Test coverage ≥ 80% (pytest)
- [ ] Ruff linting passes with no errors
- [ ] Black formatting applied
- [ ] mypy type checking passes
- [ ] No print() statements in production code
- [ ] Proper logging with appropriate levels
- [ ] Type hints on all function signatures
- [ ] Documentation strings for public APIs
```

### Python Code Quality Tools

**pyproject.toml configuration:**
```toml
[tool.black]
line-length = 100
target-version = ['py38', 'py39', 'py310', 'py311']
include = '\.pyi?$'

[tool.ruff]
line-length = 100
target-version = "py38"
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]
ignore = []

[tool.ruff.per-file-ignores]
"__init__.py" = ["F401"]

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
strict_equality = true

[tool.pytest.ini_options]
minversion = "6.0"
addopts = "-ra -q --strict-markers --cov=src --cov-report=html --cov-report=term"
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/test_*.py"]

[tool.coverage.report]
precision = 2
show_missing = true
skip_covered = false
```

### Deployment Standards

```markdown
## Pre-Deployment Checklist

### Code Quality
- [ ] All pytest tests passing
- [ ] Ruff linting passes (ruff check .)
- [ ] Black formatting applied (black --check .)
- [ ] mypy type checking passes (mypy src/)
- [ ] Test coverage ≥ 80%

### Security
- [ ] Security scan completed (no high/critical issues)
- [ ] Dependency vulnerability scan (pip-audit or safety)
- [ ] Compliance check passed
- [ ] No secrets in code or logs

### Review Process
- [ ] Code review approved (2+ reviewers)
- [ ] Architecture review for major changes
- [ ] Data impact analysis completed

### Infrastructure
- [ ] Database migrations tested
- [ ] Spark cluster configuration verified
- [ ] Resource allocation validated
- [ ] Rollback plan documented

### Monitoring
- [ ] Monitoring alerts configured
- [ ] Data quality checks in place
- [ ] Performance metrics tracked
- [ ] Runbook updated
```

### PySpark Pipeline Standards

```python
# Example pipeline with banking standards
from typing import List, Dict, Any
from pyspark.sql import SparkSession, DataFrame
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StructField, StringType, DecimalType, TimestampType
import logging

logger = logging.getLogger(__name__)


class TransactionPipeline:
    """Process banking transactions with compliance and security controls."""

    def __init__(self, spark: SparkSession):
        self.spark = spark
        self.schema = self._get_schema()

    @staticmethod
    def _get_schema() -> StructType:
        """Define strict schema for transaction data."""
        return StructType([
            StructField("transaction_id", StringType(), nullable=False),
            StructField("account_id", StringType(), nullable=False),
            StructField("amount", DecimalType(18, 2), nullable=False),
            StructField("timestamp", TimestampType(), nullable=False),
            StructField("merchant", StringType(), nullable=True),
        ])

    def validate_data(self, df: DataFrame) -> DataFrame:
        """Validate data quality and compliance rules."""
        # Check for nulls in required fields
        null_counts = df.select(
            [F.count(F.when(F.col(c).isNull(), c)).alias(c) for c in df.columns]
        )

        # Log validation metrics
        logger.info(f"Data validation: {null_counts.first()}")

        # Filter invalid records
        valid_df = df.filter(
            (F.col("transaction_id").isNotNull()) &
            (F.col("account_id").isNotNull()) &
            (F.col("amount") > 0)
        )

        return valid_df

    def mask_sensitive_data(self, df: DataFrame) -> DataFrame:
        """Mask PII for compliance (GDPR/PCI-DSS)."""
        return df.withColumn(
            "account_id_masked",
            F.concat(
                F.lit("****"),
                F.substring(F.col("account_id"), -4, 4)
            )
        )

    def process(self, input_path: str, output_path: str) -> None:
        """Execute full transaction processing pipeline."""
        try:
            # Read with strict schema
            df = self.spark.read.schema(self.schema).parquet(input_path)

            # Validate data quality
            valid_df = self.validate_data(df)

            # Apply business logic
            processed_df = self.mask_sensitive_data(valid_df)

            # Write with partitioning
            processed_df.write.mode("overwrite").partitionBy("timestamp").parquet(output_path)

            logger.info(f"Successfully processed {processed_df.count()} transactions")

        except Exception as e:
            logger.error(f"Pipeline failed: {str(e)}", exc_info=True)
            raise
```

### Incident Response

```markdown
## Using Claude Code in Incidents

1. **Investigation (Read-Only)**
   ```bash
   claude --permission-mode plan
   > Analyze error logs in /var/log/spark/
   > Search for similar issues in PySpark pipelines
   > Check data quality metrics for anomalies
   ```

2. **Fix Development (Dev Environment)**
   ```bash
   git checkout -b hotfix/issue-123
   claude
   > Fix the identified PySpark pipeline issue
   > Add pytest tests to prevent regression
   > Validate fix with sample data
   ```

3. **Documentation**
   ```bash
   > Generate incident report with timeline and root cause analysis
   > Update runbook with lessons learned
   ```

**Never:** Use Claude Code directly in production during incidents!
```

---

## Summary

In this section, you learned:

### Organization
- Naming conventions for commands and files
- Project structure best practices for Python/PySpark projects
- What to commit vs. gitignore

### Team Collaboration
- Onboarding process for Python developers
- Shared configuration with pyproject.toml
- Code review workflows with Ruff, Black, mypy, and pytest

### Security
- Least privilege principle
- Secrets management with environment variables
- Audit logging
- Regular security reviews

### Performance
- Model selection strategy
- Context management
- Using agents effectively

### Banking Standards
- Code review checklists for Python/PySpark
- Code quality tools (Ruff, Black, mypy, pytest)
- Deployment standards
- PySpark pipeline patterns
- Incident response guidelines

---

## Next Steps

1. **[Continue to Section 14: Templates Library](../06-reference/14-templates-library.md)** - Ready-to-use templates
2. **Implement these standards** - Adapt for your team
3. **Create your checklist** - Customize for your needs
4. **Set up code quality tools** - Configure Ruff, Black, mypy, and pytest
5. **Define PySpark patterns** - Establish data pipeline standards

---
