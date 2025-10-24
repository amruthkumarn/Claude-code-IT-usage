# Phase 1.3.10: Git Integration

**Learning Objectives:**
- Understand banking IT manual git policy
- Learn Claude's role in git workflows
- Master manual commit workflows
- Create pull requests with Claude's assistance
- Follow git best practices for data engineering

**Time Commitment:** 45 minutes

**Prerequisites:** Phase 1.3.1-1.3.9 completed

---

## Table of Contents
1. [Banking IT Manual Git Policy](#banking-it-manual-git-policy)
2. [Claude's Role in Git Workflow](#claudes-role-in-git-workflow)
3. [Manual Commit Workflow](#manual-commit-workflow)
4. [Manual Pull Request Workflow](#manual-pull-request-workflow)
5. [Best Practices](#best-practices)

---

## Banking IT Manual Git Policy

### 🏦 CRITICAL: No Git Automation

**ALL git operations MUST be performed manually by developers.**

This is a strict banking IT compliance requirement. Claude Code is NOT authorized to execute git commands automatically.

### What is NOT Allowed

Claude Code must **NEVER** execute:
- ❌ `git add`
- ❌ `git commit`
- ❌ `git push`
- ❌ `git pull`
- ❌ `git merge`
- ❌ `git checkout`
- ❌ `git branch`
- ❌ `gh pr create`
- ❌ Any other git write operations

### Why Manual Git Operations?

**Compliance requirements:**

1. **Audit Trail**: All git operations must be traceable to individual developers
2. **Approval Workflow**: Code changes require manual review before commit
3. **SOX Compliance**: Financial system changes require human oversight
4. **PCI-DSS**: Payment processing code requires manual verification
5. **Accountability**: Developers personally responsible for all commits

### Enforcement

Configure `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob", "Edit", "Write"],
    "deny": ["Bash"]
  }
}
```

This prevents Claude from executing **any** shell commands, including git operations.

---

## Claude's Role in Git Workflow

### What Claude CAN Do (Read-Only Assistance)

Claude helps you **understand** and **prepare** for git operations:

✅ **Suggest** commit messages (you copy and use manually)
✅ **Generate** PR descriptions (you copy to GitHub/GitLab)
✅ **Explain** git diff output (if you paste it)
✅ **Draft** release notes from commit history (if you paste it)
✅ **Help** resolve merge conflicts (you make final changes)

### What You MUST Do Manually

**All git commands are your responsibility:**

1. Review changes: `git status`, `git diff`
2. Stage changes: `git add`
3. Create commits: `git commit`
4. Push to remote: `git push`
5. Create PRs: via GitHub/GitLab UI or `gh` CLI

---

## Manual Commit Workflow

### Step 1: Development with Claude

```bash
# Start Claude Code
claude

> Implement customer transaction validation pipeline for PySpark
```

Claude makes code changes (with your approval via permission system).

### Step 2: Review Changes Manually

```bash
# Exit Claude
Ctrl+D

# Review what changed
git status
git diff

# Review specific files
git diff pipelines/validators/transaction.py
git diff tests/validators/test_transaction.py
```

### Step 3: Ask Claude for Commit Message

```bash
# Start Claude in read-only mode
claude --permission-mode plan

> I made the following changes to implement transaction validation:
>
> Files changed:
> - pipelines/validators/transaction.py (new file)
> - pipelines/schemas/transaction.py (modified)
> - tests/validators/test_transaction.py (new file)
>
> The implementation includes:
> - PySpark-based transaction validation logic
> - Schema validation for incoming data
> - Business rule validators for amount limits
> - Audit logging for validation failures
>
> Please draft a commit message following Conventional Commits format
> for our banking data engineering project
```

Claude suggests:
```
feat(pipelines): add customer transaction validation pipeline

- Implement PySpark-based transaction validation logic with DataFrame API
- Add schema validation using StructType for strict type enforcement
- Create business rule validators for amount limits and currency validation
- Add audit logging to Delta Lake table for validation failures
- Include comprehensive pytest tests with sample DataFrames

Performance: Processes 50,000 transactions/second on 4-node cluster
Compliance: Implements data quality requirement DQ-4.2
```

### Step 4: Create Commit Manually

```bash
# Exit Claude
Ctrl+D

# Stage changes
git add pipelines/validators/transaction.py
git add pipelines/schemas/transaction.py
git add tests/validators/test_transaction.py

# Commit with message (copy from Claude's suggestion)
git commit -m "feat(pipelines): add customer transaction validation pipeline

- Implement PySpark-based transaction validation logic with DataFrame API
- Add schema validation using StructType for strict type enforcement
- Create business rule validators for amount limits and currency validation
- Add audit logging to Delta Lake table for validation failures
- Include comprehensive pytest tests with sample DataFrames

Performance: Processes 50,000 transactions/second on 4-node cluster
Compliance: Implements data quality requirement DQ-4.2"
```

### Step 5: Push Manually

```bash
# Push to remote
git push origin feature/transaction-validation
```

---

## Manual Pull Request Workflow

### Step 1: Prepare PR

```bash
# Ensure all changes are committed
git status

# Check branch status
git log --oneline origin/main..HEAD

# Review full diff since branch creation
git diff main...HEAD
```

### Step 2: Ask Claude for PR Description

```bash
claude --permission-mode plan

> I have the following commits on my feature/transaction-validation branch:
>
> [paste git log output]
>
> Summary of changes:
> - Added transaction validation pipeline with PySpark
> - Created schema definitions using StructType
> - Implemented comprehensive unit tests with pytest
> - Added configuration for validation rules
> - Created pipeline documentation
>
> Please generate a comprehensive pull request description
> for our banking IT code review process. Include sections for:
> - Summary
> - Changes (file-by-file)
> - Data quality checks implemented
> - Performance characteristics
> - Compliance requirements met
> - Testing completed
> - Deployment notes
```

Claude generates:
```markdown
## Summary
Implementation of customer transaction validation pipeline using PySpark for real-time data quality checks and regulatory compliance monitoring.

## Changes

### Pipelines
- **pipelines/validators/transaction.py**: Core validation logic using PySpark DataFrame transformations
- **pipelines/schemas/transaction.py**: Schema definitions using StructType for transaction data
- **pipelines/configs/validation_rules.yaml**: Business rule configurations (amount limits, currency codes)

### Tests
- **tests/validators/test_transaction.py**: Comprehensive pytest suite with 45 test cases
  - Happy path scenarios with sample DataFrames
  - Edge cases (nulls, empty datasets, schema mismatches)
  - Data quality validation tests
  - Performance tests with 100k sample records

### Documentation
- **docs/pipelines/transaction_validation.md**: Pipeline architecture and data flow diagrams

## Data Quality Checks

### Schema Validation
- Enforces strict StructType schema for all transaction fields
- Validates data types: DecimalType for amounts, TimestampType for dates
- Checks for required fields: transaction_id, account_id, amount, timestamp

### Business Rules
- Amount limits: Validates transactions between $0.01 and $1,000,000
- Currency validation: Supports USD, EUR, GBP (ISO 4217 codes)
- Merchant verification: Validates merchant_id against reference table
- Duplicate detection: Identifies duplicate transactions within 5-minute window using window functions

### Completeness Checks
- Validates all required fields are present and non-null
- Checks referential integrity with account dimension table
- Validates timestamp ranges (not future-dated, within 90 days)

### Data Masking
- PII fields masked in logs: customer_id shows only last 4 digits
- Account numbers hashed for non-production environments
- Logging sanitized to exclude sensitive data per PCI-DSS

## Performance Characteristics

### Throughput
- Processing rate: 50,000 transactions/second on 4-node cluster (m5.2xlarge)
- Average latency: 15ms per transaction batch (10,000 records)
- Peak throughput tested: 100,000 transactions/second on 8-node cluster

### Resource Utilization
- Memory: 2GB per executor (8 executors total)
- CPU: 80% average utilization
- Shuffle operations: Optimized with broadcast joins for reference tables

### Partition Strategy
- Hash partitioning on transaction_date for optimal distribution
- 200 partitions for daily processing (~ 50MB per partition)
- Z-ORDER by account_id in Delta Lake for query performance

## Compliance

### PCI-DSS (Payment Card Industry Data Security Standard)
- **Requirement 3.4**: Data masking implemented for account_numbers in logs
- **Requirement 10.2**: Audit trail for all validation failures stored in Delta Lake
- No credit card data processed or stored

### SOX (Sarbanes-Oxley)
- **Requirement 4.2.1**: Complete audit trail for all validation failures
- Data lineage documented from source to validated output
- Immutable audit log in Delta Lake with change data feed enabled

### GDPR (General Data Protection Regulation)
- **Article 5**: Data retention policy applied (7-year financial data retention)
- Personal data encrypted at rest (AES-256)
- Right to deletion supported via soft delete flags

### BCBS 239 (Basel Committee Banking Supervision)
- Data quality metrics logged to monitoring dashboard
- Completeness, accuracy, and timeliness metrics tracked
- Monthly data quality reports generated

## Testing Completed

- [x] Unit tests passing (45 tests, 94% code coverage)
- [x] Integration tests with 100k sample dataset
- [x] Performance testing on staging cluster (4-node, 8-node)
- [x] Data quality validation report generated
- [x] Security review for PII handling completed
- [x] Ruff linting passed
- [x] Black formatting applied
- [x] mypy type checking passed

### Test Coverage by Module
- `transaction.py`: 96% coverage
- `schemas.py`: 100% coverage
- `validators/`: 92% coverage

## Deployment Notes

### Requirements
- Apache Spark 3.5+ with Python 3.10
- Delta Lake 3.0+
- AWS Glue or Databricks runtime 14.3 LTS

### Configuration
- **New file**: `config/validation_rules.yaml` → Deploy to config repository
- **Environment variables**:
  - `TRANSACTION_VALIDATION_ENABLED` (default: true)
  - `VALIDATION_BATCH_SIZE` (default: 10000)
  - `VALIDATION_CHECKPOINT_DIR` (default: s3://banking-data/checkpoints/validation)

### Database Changes
- **New Delta Lake table**: `banking.validated_transactions`
  - DDL: `ddl/validated_transactions.sql`
  - Partitioned by: transaction_date
  - Z-ORDERED by: account_id
- **New audit table**: `banking.validation_failures`
  - DDL: `ddl/validation_failures.sql`

### Monitoring
- New Grafana dashboard: `/grafana/transaction-validation`
- Metrics:
  - Validation success rate (target: >99.5%)
  - Processing throughput (transactions/second)
  - Data quality score (0-100)
  - Validation latency (p50, p95, p99)

### Rollback Plan
- Deployment uses Blue/Green strategy
- Rollback command: `databricks jobs reset --job-id 123 --version previous`
- Previous version maintained for 48 hours

## Data Flow

```
Raw Transactions (S3: s3://banking-raw/transactions/)
  ↓
Schema Validation (StructType enforcement)
  ↓
Business Rule Validation (amounts, currencies, duplicates)
  ↓
Data Quality Checks (completeness, referential integrity)
  ↓
┌─────────────────┬─────────────────┐
│                 │                 │
Validated         Rejected
Transactions      Transactions
(Delta Lake)      (Error Queue: SQS)
  ↓                 ↓
Downstream        Monitoring &
Pipelines         Alerting
```

## Reviewers
@data-engineering-lead @security-team @compliance-team @platform-team
```

### Step 3: Create PR Manually

**Option A: GitHub Web UI**
1. Navigate to GitHub repository in browser
2. Click "Pull Requests" → "New Pull Request"
3. Select your branch: `feature/transaction-validation`
4. Copy Claude's description into PR body
5. Add reviewers: data-engineering-lead, security-team, compliance-team
6. Add labels: `enhancement`, `data-pipeline`, `needs-review`
7. Click "Create Pull Request"

**Option B: GitHub CLI (Manual)**
```bash
# Save Claude's description to file
cat > pr-description.md << 'EOF'
[Paste Claude's generated description here]
EOF

# Create PR manually using gh CLI
gh pr create \
  --title "feat(pipelines): Add customer transaction validation pipeline" \
  --body-file pr-description.md \
  --reviewer data-engineering-lead,security-team,compliance-team \
  --label enhancement,data-pipeline,needs-review

# Clean up temporary file
rm pr-description.md
```

---

## Best Practices

### 1. Always Work in Feature Branches

```bash
# Create branch manually
git checkout -b feature/fraud-detection-pipeline

# Develop with Claude
claude
> Implement fraud detection pipeline using PySpark ML with Random Forest classifier

# Commit manually
Ctrl+D
git add .
git commit -m "feat(pipelines): add fraud detection ML pipeline"

# Push manually
git push origin feature/fraud-detection-pipeline
```

### 2. Review Every Change Before Committing

```bash
# Always review changes
git status
git diff

# Check specific files
git diff pipelines/ml/fraud_detection.py

# For security concerns, use Claude in plan mode
claude --permission-mode plan
> Review the fraud detection pipeline code for:
> - Security vulnerabilities
> - PII handling compliance
> - Data quality checks
> - Performance optimization opportunities
```

### 3. Use Atomic Commits

```bash
# Good: Separate commits for distinct changes
git add pipelines/etl/account_incremental_load.py
git commit -m "feat(etl): add incremental load support for accounts table"

git add tests/etl/test_account_incremental_load.py
git commit -m "test(etl): add incremental load integration tests with sample data"

git add docs/pipelines/account_incremental_load.md
git commit -m "docs(etl): document account incremental load pipeline architecture"

# Bad: Everything in one commit
git add .
git commit -m "add incremental load"  # Too vague, loses granularity
```

### 4. Leverage Claude for Commit Message Quality

```bash
# Before each commit
claude --permission-mode plan

> I changed the following files:
>
> - pipelines/processors/payment.py: Added retry logic for Kafka writes with exponential backoff
> - pipelines/validators/payment.py: Added amount range validation ($0.01 to $1M)
> - config/kafka_config.yaml: Added retry configuration (max 3 retries, 1s initial delay)
>
> Please draft a commit message following Conventional Commits format
> for our banking data engineering project. Include performance/compliance notes if relevant.
```

Claude suggests:
```
feat(pipelines): add retry logic and enhanced validation for payment processing

- Implement exponential backoff retry for Kafka writes (max 3 attempts)
- Add payment amount validation ($0.01 - $1,000,000 range)
- Configure retry policy in kafka_config.yaml (initial delay: 1s)
- Log all retry attempts to audit table for SOX compliance

Reliability: Improves payment processing success rate from 98.5% to 99.8%
Compliance: Implements retry audit requirement SOX-5.3
```

### 5. Document Git Standards in CLAUDE.md

`.claude/CLAUDE.md`:
```markdown
## Git Commit Standards

### Format: Conventional Commits
Use this format for all commits:
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat`: New feature or pipeline
- `fix`: Bug fix or data quality issue
- `refactor`: Code refactoring (no behavior change)
- `test`: Adding or updating tests
- `docs`: Documentation only
- `perf`: Performance optimization
- `chore`: Maintenance (dependencies, configs)

### Scopes (Data Engineering)
- `etl`: ETL pipelines and data ingestion
- `pipelines`: Data processing pipelines
- `validators`: Data validation and quality checks
- `transformers`: Data transformation logic
- `ml`: Machine learning models and training
- `schemas`: Data schemas and table DDL
- `config`: Configuration files
- `infra`: Infrastructure as code (Terraform, CloudFormation)

### Example: Pipeline Feature
```
feat(pipelines): add real-time fraud detection pipeline

- Implement streaming pipeline with PySpark Structured Streaming
- Add Random Forest ML model for fraud score calculation (AUC: 0.94)
- Create alert generation for high-risk transactions (score > 0.8)
- Add monitoring metrics for model performance drift detection
- Include pytest tests with synthetic fraud patterns

Performance: Processes 10,000 events/second with <100ms latency
Compliance: Implements fraud detection requirement FD-3.1
Model: Random Forest (100 trees, max_depth=10, AUC=0.94)
```

### Example: Bug Fix
```
fix(validators): prevent false positives in duplicate transaction detection

- Fix window function logic to use transaction_timestamp instead of processing_time
- Add merchant_id to duplicate detection key (transaction_id alone insufficient)
- Update unit tests with edge cases for same-second transactions

Issue: Transactions within same second from different merchants flagged as duplicates
Impact: Reduces false positive rate from 2.3% to 0.1%
```

### Example: Performance Optimization
```
perf(etl): optimize customer aggregation pipeline with broadcast join

- Replace sort-merge join with broadcast join for customer dimension (10MB table)
- Add caching for frequently-reused intermediate DataFrames
- Increase partition count from 200 to 400 for better parallelism
- Add Z-ORDER by customer_id in Delta Lake output table

Performance: Reduces runtime from 45 minutes to 12 minutes (73% improvement)
Resource: Reduces shuffle data from 50GB to 5GB
```
```

### 6. Use Pre-Commit Hooks (Optional)

Set up git hooks to run before commit (validation only, not blocking):

`.git/hooks/pre-commit`:
```bash
#!/bin/bash
# Pre-commit validation for data pipelines

echo "Running pre-commit checks..."

# Run Black formatter check
echo "Checking code formatting..."
python -m black --check . || {
  echo "❌ Formatting issues found. Run: black ."
  exit 1
}

# Run Ruff linter
echo "Running Ruff linter..."
python -m ruff check . || {
  echo "❌ Linting issues found. Run: ruff check --fix ."
  exit 1
}

# Run mypy type checker
echo "Running mypy type checker..."
python -m mypy pipelines/ --ignore-missing-imports || {
  echo "❌ Type checking failed. Fix type hints."
  exit 1
}

# Run pytest (fast tests only)
echo "Running unit tests..."
python -m pytest tests/unit -m "not slow" || {
  echo "❌ Unit tests failed"
  exit 1
}

# Check for secrets
echo "Checking for hardcoded secrets..."
if grep -rE '(password|secret|api_key|token)\s*=\s*["\047][^"\047]{8,}' pipelines/; then
  echo "❌ Potential hardcoded secrets detected"
  exit 1
fi

echo "✅ All pre-commit checks passed"
exit 0
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

---

## Common Workflows

### Workflow 1: Pipeline Development

```bash
# 1. Create branch manually
git checkout -b feature/daily-balance-aggregation

# 2. Develop with Claude
claude
> Implement daily account balance aggregation pipeline using PySpark
> Include:
> - Read from Delta Lake accounts table
> - Aggregate by account_id and date
> - Calculate running balance with window functions
> - Write to Delta Lake with partitioning by date
> - Add comprehensive pytest tests

# Claude creates pipeline, tests, and documentation

# 3. Exit and review
Ctrl+D
git status
git diff

# 4. Get commit message from Claude
claude --permission-mode plan
> Draft commit message for daily balance aggregation pipeline implementation

# 5. Commit manually
git add pipelines/aggregations/daily_balance.py
git add tests/aggregations/test_daily_balance.py
git add docs/pipelines/daily_balance.md
git commit -m "[paste Claude's message]"

# 6. Push manually
git push origin feature/daily-balance-aggregation

# 7. Get PR description from Claude
claude --permission-mode plan
> Generate comprehensive PR description for daily balance aggregation pipeline

# 8. Create PR manually via GitHub UI or gh CLI
gh pr create --title "feat(pipelines): Add daily account balance aggregation" \
  --body-file pr-description.md
```

### Workflow 2: Bug Fix

```bash
# 1. Create branch manually
git checkout -b fix/duplicate-transaction-handling

# 2. Fix with Claude
claude
> Fix the duplicate transaction detection logic in pipelines/validators/transaction.py
>
> Issue: Transactions within the same second from different merchants
> are being flagged as duplicates. The current logic only uses transaction_id
> for duplicate detection, but merchant_id should also be part of the key.

# 3. Review and commit manually
Ctrl+D
git diff pipelines/validators/transaction.py
git diff tests/validators/test_transaction.py

git add pipelines/validators/transaction.py tests/validators/test_transaction.py
git commit -m "fix(validators): prevent false positives in duplicate detection

- Add merchant_id to duplicate detection key
- Fix window function to use transaction_timestamp
- Add test cases for same-second transactions

Impact: Reduces false positive rate from 2.3% to 0.1%"

# 4. Push manually
git push origin fix/duplicate-transaction-handling

# 5. Create PR manually
gh pr create --title "fix: Prevent false positives in duplicate transaction detection"
```

### Workflow 3: Code Review Assistance

```bash
# You receive PR review comments
# Use Claude to help address them

claude
> I received this review comment on PR #456:
>
> "The partition strategy in customer_aggregation.py could be optimized.
> Currently using hash partitioning on customer_id, but the downstream
> queries primarily filter by date. Consider partition by date and
> Z-ORDER by customer_id in Delta Lake."
>
> Please improve the partitioning logic in pipelines/aggregations/customer_aggregation.py

# Claude refactors the partitioning strategy

# Review and commit manually
Ctrl+D
git add pipelines/aggregations/customer_aggregation.py
git commit -m "perf(aggregations): optimize customer aggregation partitioning

- Change from hash partition by customer_id to partition by date
- Add Z-ORDER by customer_id in Delta Lake for query performance
- Update tests to verify partitioning strategy

Performance: Reduces query time from 5s to 0.8s for date-filtered queries"

git push
```

---

## Configuration for Banking IT

### Recommended Settings

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Grep", "Glob"],
    "requireApproval": [],
    "deny": ["Bash", "WebFetch", "WebSearch"]
  },

  "defaultModel": "sonnet",

  "env": {
    "GIT_OPERATIONS": "MANUAL_ONLY",
    "PYSPARK_PYTHON": "python3"
  }
}
```

### Project Memory

`.claude/CLAUDE.md`:
```markdown
# Git Operations Policy

## CRITICAL: Manual Git Only

ALL git operations MUST be performed manually by developers.

### When Asked to Commit or Push

1. **Suggest** commit message following Conventional Commits
2. **Suggest** git commands to execute
3. **NEVER** execute git commands
4. **Remind** user to perform git operations manually

### Workflow

1. Claude makes code changes (with permission approval)
2. Developer reviews changes manually (`git diff`)
3. Developer asks Claude for commit message (plan mode)
4. Developer executes git commands manually
5. Developer creates PR manually (can use Claude's description)

## Commit Message Format for Data Engineering

### Pipeline Changes
```
feat(pipelines): add customer account aggregation pipeline

- Implement daily batch aggregation using PySpark DataFrame API
- Add data quality checks for account balance consistency
- Create monitoring metrics for pipeline performance (throughput, latency)
- Add error handling with retry logic and dead letter queue
- Include pytest tests with sample DataFrames (100k records)

Performance: Processes 10M accounts in 15 minutes (11k accounts/second)
Compliance: Implements SOX audit requirement 4.2.1
```

### Schema Changes
```
feat(schemas): add transaction enrichment fields to payment schema

- Add merchant_category field (StringType) to transaction StructType
- Add risk_score calculated field (DecimalType(5,2), range 0-100)
- Update Delta Lake table DDL with new columns
- Create backfill job for existing data (run_backfill_enrichment.py)

Migration: Requires one-time backfill job (est. 6 hours for 5B records)
Impact: Enables fraud detection and merchant analytics use cases
```

### Bug Fixes
```
fix(validators): correct null handling in payment amount validation

- Fix NullPointerException when amount field is null
- Add explicit null check before Decimal comparison
- Update schema to make amount field non-nullable (StructField nullable=False)
- Add test cases for null amount scenarios

Issue: Pipeline crashed on 0.1% of records with null amounts
Impact: Eliminates validation pipeline failures, improves stability
```
```

---

## Summary

In this subsection, you learned:

### Banking IT Git Policy
- ✅ Claude helps write PySpark code and data pipelines
- ✅ Claude suggests commit messages and PR descriptions
- ✅ Claude generates documentation and analysis
- ❌ Claude NEVER executes git commands
- ✅ All git operations performed manually by developers

### Developer Workflow
1. **Develop**: Work with Claude to write/modify data pipelines
2. **Review**: Manually review changes with `git diff`
3. **Draft**: Ask Claude for commit message/PR description (plan mode)
4. **Commit**: Manually execute git commands
5. **Push**: Manually push to remote
6. **PR**: Manually create pull request with Claude's description

### Key Benefits
- Maintains regulatory audit trail and accountability
- Complies with SOX, PCI-DSS, GDPR requirements
- Prevents accidental commits of sensitive data
- Developer retains full control over version history
- Leverages Claude for high-quality commit messages and data engineering best practices

---

## Next Steps

👉 **[Continue to 1.3.11: Standards & Best Practices](./03-11-standards-best-practices.md)**

**Quick Practice:**
1. Configure `.claude/settings.json` to deny Bash (enforce manual git)
2. Develop a small PySpark transformation with Claude
3. Ask Claude for commit message in plan mode
4. Execute git commands manually

---

**Related Sections:**
- [Phase 1.3.5: Project Configuration](./03-05-project-configuration.md) - Settings configuration
- [Phase 1.3.8: Hooks & Automation](./03-08-hooks-automation.md) - Automation policies
- [Phase 1.3.11: Standards & Best Practices](./03-11-standards-best-practices.md) - Code standards

---

**Last Updated:** 2025-10-24
**Version:** 1.0
