# Section 12: Version Control Integration

## Table of Contents
1. [Git with Claude Code](#git-with-claude-code)
2. [Banking IT Policy: Manual Git Operations](#banking-it-policy-manual-git-operations)
3. [Claude's Role in Git Workflow](#claudes-role-in-git-workflow)
4. [Manual Commit Workflow](#manual-commit-workflow)
5. [Manual Pull Request Workflow](#manual-pull-request-workflow)
6. [Best Practices](#best-practices)

---

## Git with Claude Code

### 🏦 Banking IT Policy

**IMPORTANT: Git automation via Claude Code is NOT approved for banking environments.**

All git operations (commit, push, pull, merge) MUST be performed manually by developers. This section documents how to work with Claude Code while maintaining manual control of all version control operations.

---

## Banking IT Policy: Manual Git Operations

### What is NOT Allowed

Claude Code must **NEVER** execute the following commands:
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

Banking IT requirements:
1. **Audit Trail**: All git operations must be traceable to individual developers
2. **Approval Workflow**: Code changes require manual review before commit
3. **Compliance**: SOX, PCI-DSS regulations require human oversight
4. **Security**: Prevent accidental commits of sensitive data
5. **Accountability**: Developers are personally responsible for commits

### Enforcement

Configure `.claude/settings.json` to prevent git automation:

```json
{
  "permissions": {
    "deny": ["Bash"]
  }
}
```

This prevents Claude from executing any shell commands, including git operations.

---

## Claude's Role in Git Workflow

### What Claude CAN Do (Read-Only)

Claude can help you **understand** your git state:
- ✅ **Suggest** commit messages (you copy and use manually)
- ✅ **Generate** PR descriptions (you copy to GitHub/GitLab)
- ✅ **Explain** git diff output (if you paste it)
- ✅ **Draft** release notes from commit history (if you paste it)
- ✅ **Help** resolve merge conflicts (you make final changes)

### What You MUST Do Manually

All git commands are your responsibility:
1. Review changes with `git status` and `git diff`
2. Stage changes with `git add`
3. Create commits with `git commit`
4. Push to remote with `git push`
5. Create PRs via GitHub/GitLab UI or `gh` CLI manually

---

## Manual Commit Workflow

### Step 1: Development with Claude

```bash
# Work with Claude
claude

> Implement customer transaction validation pipeline
```

Claude makes code changes (with your approval).

### Step 2: Review Changes Manually

```bash
# Exit Claude
Ctrl+D

# Review what changed
git status
git diff

# Review specific files
git diff pipelines/validators/transaction.py
```

### Step 3: Ask Claude for Commit Message

```bash
# Start Claude in read-only mode
claude --permission-mode plan

> I made the following changes:
> [paste git diff or describe changes]
>
> Please draft a commit message following Conventional Commits format
> for our banking data engineering project
```

Claude suggests:
```
feat(pipelines): add customer transaction validation pipeline

- Implement PySpark-based transaction validation logic
- Add schema validation for incoming transaction data
- Create business rule validators for amount limits
- Add audit logging for validation failures

Compliance: Implements data quality requirement DQ-4.2
```

### Step 4: Create Commit Manually

```bash
# Exit Claude
Ctrl+D

# Stage changes
git add pipelines/validators/transaction.py pipelines/schemas/transaction.py tests/validators/test_transaction.py

# Commit with message (copy from Claude's suggestion)
git commit -m "feat(pipelines): add customer transaction validation pipeline

- Implement PySpark-based transaction validation logic
- Add schema validation for incoming transaction data
- Create business rule validators for amount limits
- Add audit logging for validation failures

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
```

### Step 2: Ask Claude for PR Description

```bash
claude --permission-mode plan

> I have the following commits on my branch:
> [paste git log output]
>
> Please generate a comprehensive pull request description
> for our banking IT code review process
```

Claude generates:
```markdown
## Summary
Implementation of customer transaction validation pipeline with PySpark for real-time data quality checks and compliance monitoring.

## Changes
- **pipelines/validators/transaction.py**: Transaction validation logic with PySpark transformations
- **pipelines/schemas/transaction.py**: Schema definitions for transaction data (Spark StructType)
- **tests/validators/test_transaction.py**: Comprehensive unit tests using pytest and pyspark.testing
- **config/validation_rules.yaml**: Business rule configurations for transaction validation
- **docs/pipelines/validation.md**: Pipeline documentation and data flow diagrams

## Data Quality Checks
- Schema validation: Enforces strict data types for all transaction fields
- Business rules: Amount limits, currency validation, merchant verification
- Duplicate detection: Identifies duplicate transactions within 5-minute window
- Completeness checks: Validates required fields are present and non-null
- Data masking: PII fields masked in logs (customer_id, account_number)

## Performance Characteristics
- Processing throughput: 50,000 transactions/second on 4-node cluster
- Latency: Average validation time 15ms per transaction batch
- Resource utilization: 2GB memory per executor, 80% CPU efficiency
- Partition strategy: Hash partitioning on transaction_date for optimal distribution

## Compliance
- **SOX**: Audit trail for all validation failures (requirement 4.2.1)
- **PCI-DSS**: Data masking for sensitive fields (requirement 3.4)
- **GDPR**: Data retention policy applied (7-year transaction history)
- **BCBS 239**: Data quality metrics logged to monitoring dashboard

## Testing Completed
- [x] Unit tests passing (156 tests, 94% coverage)
- [x] Integration tests with sample datasets (10M records)
- [x] Performance testing on staging cluster
- [x] Data quality validation report generated
- [x] Security review for PII handling completed

## Deployment Notes
- Requires Apache Spark 3.5+ with Python 3.10
- New configuration: `config/validation_rules.yaml` (deploy to config repository)
- Delta Lake table: `banking.validated_transactions` (DDL included in migration)
- Monitoring: New dashboard at /grafana/transaction-validation
- Environment variables:
  - `TRANSACTION_VALIDATION_ENABLED` (default: true)
  - `VALIDATION_BATCH_SIZE` (default: 10000)
  - `VALIDATION_CHECKPOINT_DIR` (default: s3://banking-data/checkpoints/validation)

## Data Flow
```
Raw Transactions (S3)
  → Schema Validation
  → Business Rule Validation
  → Duplicate Detection
  → Validated Transactions (Delta Lake)
  → Rejected Transactions (Error Queue)
```

## Reviewers
@data-engineering-lead @security-team @compliance-team
```

### Step 3: Create PR Manually

**Option A: GitHub Web UI**
1. Go to GitHub repository
2. Click "New Pull Request"
3. Select your branch
4. Copy Claude's description into PR body
5. Add reviewers
6. Create PR

**Option B: GitHub CLI (Manual)**
```bash
# Create PR manually using gh CLI
gh pr create \
  --title "feat(pipelines): Add customer transaction validation pipeline" \
  --body-file pr-description.md \
  --reviewer @data-engineering-lead,@security-team,@compliance-team

# Where pr-description.md contains Claude's generated description
```

---

## Best Practices

### 1. Always Work in Feature Branches

```bash
# Create branch manually
git checkout -b feature/fraud-detection-pipeline

# Develop with Claude
claude
> Implement fraud detection pipeline using PySpark ML

# Commit manually (as described above)
git add .
git commit -m "feat(pipelines): add fraud detection ML pipeline"

# Push manually
git push origin feature/fraud-detection-pipeline
```

### 2. Review Every Change Before Committing

```bash
# Always review
git status
git diff

# For specific concerns, ask Claude
claude --permission-mode plan
> Review the changes in pipelines/transformers/payment.py for data security issues
```

### 3. Use Atomic Commits

```bash
# Good: Separate commits for separate concerns
git add pipelines/etl/
git commit -m "feat(etl): add incremental load support for accounts"

git add tests/etl/
git commit -m "test(etl): add incremental load integration tests"

# Bad: Everything in one commit
git add .
git commit -m "add incremental load"
```

### 4. Leverage Claude for Commit Message Quality

```bash
# Before each commit, ask Claude
claude --permission-mode plan

> I changed the following files:
> - pipelines/processors/payment.py: Add retry logic for Kafka writes
> - pipelines/validators/payment.py: Add amount range validation
>
> Draft a commit message following our Conventional Commits standard
```

### 5. Document Git Standards in CLAUDE.md

`.claude/CLAUDE.md`:
```markdown
## Git Commit Standards

### Format
Use Conventional Commits format:
- `feat(scope)`: New feature or pipeline
- `fix(scope)`: Bug fix or data quality issue
- `refactor(scope)`: Code refactoring
- `test(scope)`: Adding tests
- `docs(scope)`: Documentation
- `perf(scope)`: Performance optimization
- `chore(scope)`: Maintenance

### Example
```
feat(pipelines): add real-time fraud detection pipeline

- Implement streaming pipeline with PySpark Structured Streaming
- Add ML model for fraud score calculation (Random Forest)
- Create alert generation for high-risk transactions
- Add monitoring metrics for model performance

Compliance: Implements fraud detection requirement FD-3.1
```

### Scopes
- etl: ETL pipelines and data ingestion
- pipelines: Data processing pipelines
- validators: Data validation and quality checks
- transformers: Data transformation logic
- ml: Machine learning models and training
- schemas: Data schemas and table definitions
- config: Configuration files
- infra: Infrastructure as code
```

### 6. Use Pre-Commit Hooks (Manual)

Set up git hooks to run before commit:

`.git/hooks/pre-commit`:
```bash
#!/bin/bash
# Run Python linting
poetry run black --check . || exit 1
poetry run ruff check . || exit 1

# Run tests
poetry run pytest tests/ || exit 1

# Check for secrets and credentials
./scripts/detect-secrets.sh || exit 1

# Validate PySpark code
poetry run mypy pipelines/ || exit 1
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
git checkout -b feature/daily-aggregation-pipeline

# 2. Develop with Claude
claude
> Implement daily account balance aggregation pipeline
> Add tests for aggregation logic

# 3. Exit and review
Ctrl+D
git status
git diff

# 4. Get commit message from Claude
claude --permission-mode plan
> Draft commit message for daily aggregation pipeline implementation

# 5. Commit manually
git add .
git commit -m "[paste Claude's suggested message]"

# 6. Push manually
git push origin feature/daily-aggregation-pipeline

# 7. Get PR description from Claude
claude --permission-mode plan
> Generate PR description for daily aggregation pipeline

# 8. Create PR manually via GitHub UI
```

### Workflow 2: Bug Fix

```bash
# 1. Create branch manually
git checkout -b fix/duplicate-transaction-handling

# 2. Fix with Claude
claude
> Fix the duplicate transaction detection logic in pipelines/validators/transaction.py

# 3. Review and commit manually
Ctrl+D
git diff
git add pipelines/validators/transaction.py
git commit -m "fix(validators): prevent false positives in duplicate detection"

# 4. Push manually
git push origin fix/duplicate-transaction-handling

# 5. Create PR manually
```

### Workflow 3: Code Review Assistance

```bash
# You receive PR review comments
# Use Claude to help address them

claude
> I received this review comment on PR #456:
> "The partition strategy in customer_pipeline.py could be optimized for better performance"
>
> Please improve the partitioning logic in pipelines/etl/customer_pipeline.py

# Claude makes changes

# Review and commit manually
Ctrl+D
git add pipelines/etl/customer_pipeline.py
git commit -m "perf(etl): optimize customer pipeline partitioning strategy"
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
    "deny": ["Bash"]
  },

  "defaultModel": "sonnet",

  "env": {
    "GIT_OPERATIONS": "MANUAL_ONLY"
  }
}
```

### Project Memory

`.claude/CLAUDE.md`:
```markdown
# Git Operations Policy

## CRITICAL: Manual Git Only

ALL git operations MUST be performed manually by developers.

When asked to commit or push:
1. Suggest commit message
2. Suggest git commands
3. NEVER execute git commands
4. Remind user to perform git operations manually

## Workflow
1. Claude makes code changes (with approval)
2. Developer reviews changes manually (git diff)
3. Developer asks Claude for commit message
4. Developer executes git commands manually
5. Developer creates PR manually (can use Claude's description)

## Commit Message Format for Data Engineering

### Pipeline Changes
```
feat(pipelines): add customer account aggregation pipeline

- Implement daily batch aggregation using PySpark
- Add data quality checks for account balances
- Create monitoring metrics for pipeline performance
- Add error handling and retry logic

Performance: Processes 10M accounts in 15 minutes
Compliance: Implements requirement DQ-5.3
```

### Schema Changes
```
feat(schemas): add transaction enrichment fields

- Add merchant_category field to transaction schema
- Add risk_score calculated field
- Update Delta Lake table DDL
- Migrate existing data with backfill job

Migration: Requires backfill job (run_backfill_enrichment.py)
```
```

---

## Summary

### Banking IT Git Policy
- ✅ Claude helps write PySpark code
- ✅ Claude suggests commit messages
- ✅ Claude generates PR descriptions
- ❌ Claude NEVER executes git commands
- ✅ All git operations performed manually by developers

### Developer Workflow
1. **Develop**: Work with Claude to write/modify pipelines
2. **Review**: Manually review changes with `git diff`
3. **Draft**: Ask Claude for commit message/PR description
4. **Commit**: Manually execute git commands
5. **Push**: Manually push to remote
6. **PR**: Manually create pull request

### Key Benefits
- Maintains audit trail and accountability
- Complies with banking regulations
- Prevents accidental commits of sensitive data
- Developer retains full control
- Leverages Claude for quality commit messages and data engineering best practices

---

## Next Steps

1. **[Continue to Section 13: Standards & Best Practices](./13-standards-best-practices.md)** - Team standards
2. **Configure your project** - Set up `.claude/settings.json` to deny Bash
3. **Update CLAUDE.md** - Document manual git policy for data engineering workflows

---

**Policy Status:** Manual Git Operations REQUIRED
