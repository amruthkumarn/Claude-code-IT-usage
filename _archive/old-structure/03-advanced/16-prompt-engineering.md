# Section 16: Prompt Engineering for Claude Code

## Table of Contents
1. [Introduction to Prompt Engineering](#introduction-to-prompt-engineering)
2. [Fundamentals](#fundamentals)
3. [Intermediate Techniques](#intermediate-techniques)
4. [Advanced Techniques](#advanced-techniques)
5. [Banking-Specific Prompts](#banking-specific-prompts)
6. [Prompt Templates Library](#prompt-templates-library)
7. [Best Practices](#best-practices)

---

## Introduction to Prompt Engineering

**Prompt engineering** is the art and science of crafting effective instructions for AI models like Claude. In Claude Code, good prompts lead to:
- More accurate code generation
- Better analysis and insights
- Fewer iterations to get desired results
- Consistent, high-quality outputs

### Why Prompt Engineering Matters in Banking IT

In banking environments, prompt quality directly impacts:
- **Accuracy**: Financial calculations must be precise
- **Compliance**: Security and regulatory requirements must be met
- **Auditability**: Actions must be clear and traceable
- **Efficiency**: Time saved with better first attempts

### Learning Resources

This section is based on **Anthropic's Interactive Prompt Engineering Tutorial**:
- **GitHub**: https://github.com/anthropics/prompt-eng-interactive-tutorial
- **Official Docs**: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

---

## Fundamentals

### 1. Basic Prompt Structure

A good prompt has three components:

```
[TASK] + [CONTEXT] + [FORMAT]
```

**Bad Prompt:**
```
> Fix the bug
```

**Good Prompt:**
```
> The authentication validation pipeline returns errors when the password field is missing.
> This is in pipelines/validators/authentication.py.
> Please:
> 1. Add input validation
> 2. Return appropriate error with clear message
> 3. Add a test case
```

### 2. Being Clear and Direct

**Vague:**
```
> Make this code better
```

**Clear:**
```
> Refactor pipelines/payments/processor.py to:
> - Use PySpark DataFrame operations instead of RDD transformations
> - Add error handling for schema mismatches
> - Extract validation logic into separate functions
> - Add docstrings with type hints
```

### 3. Providing Context

**Without Context:**
```
> Add authentication
```

**With Context:**
```
> Add IAM role-based authentication to the data pipeline access.

Requirements:
- Use the existing UserService for validation
- Session expiry: 15 minutes
- Include user role in session metadata
- Follow the pattern in utils/validators/auth.py
```

### 4. Assigning Roles (Using CLAUDE.md)

Instead of repeating context every time, use CLAUDE.md:

```markdown
# Banking Data Engineering Project

## Your Role
You are a senior banking data engineer with expertise in:
- Payment data processing pipelines
- PCI-DSS compliance for data systems
- Python 3.9+ and PySpark
- Data security best practices

## Always Remember
- All amounts in cents (integers, never floats)
- Validate input against Pydantic models or StructType schemas
- Use parameterized Spark SQL queries
- Log security events (but never log passwords/tokens)
- Follow PCI-DSS requirements for data pipelines
```

Now prompts can be simpler:
```
> Add a payment data transformation pipeline
```

Claude will automatically apply banking context!

---

## Intermediate Techniques

### 1. Separating Data from Instructions

Use clear delimiters to separate instructions from data:

**Using XML Tags:**
```
> Analyze this code for Spark SQL injection vulnerabilities:

<code>
query = f"SELECT * FROM users WHERE id = {user_id}"
spark.sql(query)
</code>

Check:
- Are queries parameterized?
- Is input validated?
- Are there any string f-string injections?
```

**Using Markdown:**
```
> Review the following function:

```python
def process_payment(amount, card_number):
    logger.info(f'Processing payment: {amount}, {card_number}')
    return api.charge(amount, card_number)
```

Issues to check:
- PCI-DSS compliance
- Error handling
- Logging sensitive data
```

### 2. Formatting Output

Request specific formats for better integration:

**JSON Output:**
```
> Scan this codebase for hardcoded secrets.
> Return results as JSON array with this structure:

{
  "findings": [
    {
      "file": "path/to/file.py",
      "line": 42,
      "type": "api_key",
      "severity": "high",
      "value": "AKIA..."
    }
  ]
}
```

**Table Format:**
```
> List all data pipeline jobs in this project.
> Format as markdown table:

| Pipeline Name | Source | Destination | Auth | Description |
|---------------|--------|-------------|------|-------------|
```

**Structured Report:**
```
> Perform security audit and return report in this format:

## Summary
[One paragraph overview]

## Critical Issues
- [Issue 1]
- [Issue 2]

## Recommendations
1. [Action item with code example]
2. [Action item with code example]
```

### 3. Chain of Thought (Step-by-Step Reasoning)

For complex problems, ask Claude to think step-by-step:

```
> Debug why the DataFrame transaction rollback is not working.

Think through this step by step:
1. First, identify where Spark transactions are initiated
2. Then, trace the error handling path
3. Check if rollback logic is in exception blocks
4. Verify Spark session configuration
5. Finally, suggest the fix with code
```

**Example:**
```
> Calculate compound interest for a savings account.

Let's solve this step by step:
1. Principal: $10,000
2. Rate: 5% annual
3. Compound: Monthly
4. Duration: 2 years

Show your calculation for each month, then give final amount.
```

### 4. Using Examples (Few-Shot Learning)

Provide examples of desired output:

```
> Generate validation schemas for these data pipeline inputs.

Example format:

# Pipeline: payment_ingestion
from pydantic import BaseModel, Field

class PaymentSchema(BaseModel):
    email: str = Field(..., pattern=r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
    amount: int = Field(..., gt=0)
    account_type: str = Field('standard', pattern='^(standard|premium)$')

Now create schemas for:
- transaction_processing pipeline
- account_aggregation pipeline
- balance_update pipeline
```

**Banking Example:**
```
> Generate test cases for payment data validation.

Example test case:

def test_reject_payment_with_insufficient_funds():
    account = create_test_account(balance=100)
    payment_data = {'amount': 200, 'account_id': account.id}

    with pytest.raises(ValueError, match='Insufficient funds'):
        process_payment(payment_data)

Create similar tests for:
- Invalid account ID
- Negative amount
- Amount exceeds daily limit
- Duplicate transaction (idempotency)
```

---

## Advanced Techniques

### 1. Avoiding Hallucinations

**Problem**: Claude might invent functions or APIs that don't exist.

**Solution**: Ground Claude in actual code.

**Bad:**
```
> Add error handling to the payment processor
```

**Good:**
```
> Add error handling to the payment data processor.

Current code structure:
- PaymentProcessor class in pipelines/payments/processor.py
- Uses PaymentGatewayAPI from integrations/payment_gateway.py
- Error types defined in errors/payment_errors.py

Add handling for:
- Network timeouts (use existing TimeoutError)
- Invalid card data (use existing CardError)
- Insufficient funds (create new InsufficientFundsError)

Follow the error handling pattern used in services/account_service.py
```

### 2. Verification Prompts

Ask Claude to verify its own work:

```
> After implementing the password reset feature, verify:

Checklist:
- [ ] Token expiry is enforced (max 1 hour)
- [ ] Old token is invalidated after use
- [ ] Password meets complexity requirements
- [ ] Email is sent over secure connection
- [ ] No sensitive data in logs
- [ ] Rate limiting is applied

For each item, show the relevant code that implements it.
```

### 3. Multi-Step Complex Prompts

Break complex tasks into orchestrated steps:

```
> I need to add two-factor authentication (2FA).

Phase 1: Database Schema
- Review existing users table
- Design 2FA fields (secret, backup codes, enabled flag)
- Generate migration script

Phase 2: Backend Pipeline
- Add validation step to enable 2FA
- Add validation step to verify TOTP code
- Modify authentication pipeline to check 2FA status

Phase 3: Testing
- Unit tests for TOTP generation/validation
- Integration tests for 2FA flow
- Test backup codes

Let's start with Phase 1. After I approve it, we'll move to Phase 2.
```

### 4. Iterative Refinement

Use follow-up prompts to refine:

```
First prompt:
> Generate a function to calculate late payment fees

Second prompt (refine):
> Good start. Now add:
> - Cap maximum fee at $50
> - Grace period of 3 days
> - Compound fees monthly (not daily)
> - Add docstrings with type hints

Third prompt (final):
> Perfect. Now add test cases covering:
> - Payment 1 day late
> - Payment 10 days late
> - Payment 60 days late (test cap)
```

---

## Banking-Specific Prompts

### Security Review Prompt

```
> Perform comprehensive security review of pipelines/payments/

Check for:

## Authentication & Authorization
- All pipeline jobs require proper IAM authentication?
- Proper role-based access control?
- Data access validation correct?

## Input Validation
- Spark SQL injection prevention?
- Schema validation before processing?
- Amount validation (positive, max limits)?

## PCI-DSS Compliance
- No credit card numbers in logs?
- No CVV storage?
- Encryption at rest and in transit?

## Financial Accuracy
- Amounts in cents (integers)?
- Transaction atomicity in DataFrame operations?
- Audit logging?

For each issue found, provide:
- File and line number
- Severity (Critical/High/Medium/Low)
- Code example of the issue
- Recommended fix with code
```

### Compliance Check Prompt

```
> Audit this codebase for regulatory compliance.

Standards to check:

## PCI-DSS
- Cardholder data storage in DataFrames
- Encryption requirements
- Access controls
- Logging requirements

## SOX
- Financial transaction audit trails
- Change management controls
- Separation of duties

## GDPR
- Personal data identification in datasets
- Consent tracking
- Right to deletion
- Data retention policies

Generate report with:
- Compliance status per requirement
- Non-compliant code locations
- Remediation steps
- Priority ranking
```

### Code Generation with Banking Context

```
> Generate a payment data transformation pipeline.

Requirements:
- Pipeline: payment_processing
- Input: { account_id, amount, description, idempotency_key }
- Validate:
  * Account exists and is active
  * Sufficient balance
  * Amount > 0 and <= $10,000 (daily limit)
  * Idempotency key prevents duplicates
- Transformation:
  * Debit account
  * Create payment record
  * Create audit log entry
- Output: Processed payment records with status
- Errors: ValueError (validation), InsufficientFundsError, DuplicateTransactionError

Use existing:
- AccountService (services/account_service.py)
- PaymentRepository (repositories/payment_repository.py)
- AuditLogger (utils/audit_logger.py)

Follow patterns in pipelines/transactions/transformations.py
```

### Debugging Prompt with Context

```
> DataFrame transaction processing is failing intermittently.

Symptoms:
- About 5% of failed payments leave orphaned records
- Happens more during high load
- Error logs show "executor memory exhausted"

Investigation needed:
1. Check transformation boundaries in PaymentProcessor
2. Verify Spark session configuration
3. Look for missing error handling
4. Check for memory-intensive operations
5. Identify any data skew conditions

Codebase:
- Payment processing: pipelines/payments/processor.py
- Spark config: config/spark_config.py
- Transaction util: utils/transaction_handler.py

Debug step by step and explain findings.
```

---

## Prompt Templates Library

### Template 1: Feature Development

```
> Implement [FEATURE_NAME]

Requirements:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

Technical details:
- Technology: [Python/PySpark]
- Integration: [Existing components]
- Data model: [Schema details]

Acceptance criteria:
- [ ] [Criteria 1]
- [ ] [Criteria 2]
- [ ] [Criteria 3]

Please:
1. Design the solution
2. Implement with tests
3. Add documentation
4. Consider edge cases
```

### Template 2: Code Review

```
> Review [FILE_OR_DIRECTORY]

Focus areas:
- Security vulnerabilities
- Performance issues (DataFrame operations, partitioning)
- Code quality
- Best practices
- Banking compliance

For each finding, provide:
- Location (file:line)
- Issue description
- Severity
- Recommended fix
- Code example

Prioritize critical security and compliance issues.
```

### Template 3: Bug Fix

```
> Fix bug: [DESCRIPTION]

Error details:
- Symptom: [What's happening]
- Expected: [What should happen]
- Location: [Where it occurs]
- Logs: [Error messages]

Context:
- Affected file(s): [Paths]
- Related code: [Dependencies]
- Recent changes: [Commits]

Please:
1. Identify root cause
2. Propose fix
3. Add regression test
4. Consider edge cases
```

### Template 4: Test Generation

```
> Generate comprehensive tests for [FILE/FUNCTION]

Test categories:
- Happy path (success cases)
- Edge cases
- Error conditions
- Boundary values
- Security scenarios

For each test:
- Descriptive test name
- Arrange-Act-Assert pattern
- Mock external dependencies
- Use banking test data (never real)

Framework: pytest
Coverage target: 100% for critical paths
```

### Template 5: Documentation

```
> Generate documentation for [COMPONENT]

Include:
- Overview and purpose
- API reference (functions/methods)
- Parameters and return types (with type hints)
- Usage examples
- Error handling
- Banking-specific notes
- Security considerations

Format: Markdown with Python docstrings
Audience: [Developers/Operations/etc]
```

### Template 6: Migration Script

```
> Create database migration for [CHANGE]

Details:
- Database: [PostgreSQL/MySQL/etc]
- Change type: [ADD/MODIFY/DELETE]
- Tables affected: [Table names]

Requirements:
- Transaction wrapped
- Rollback script (DOWN migration)
- Data migration if needed
- Indexes for new foreign keys
- Comments explaining changes

Consider:
- Data integrity
- Downtime requirements
- Backup strategy
- Rollback plan
```

---

## Best Practices

### 1. Be Specific, Not General

**General:**
```
> Improve this code
```

**Specific:**
```
> Refactor get_user_transactions() to:
> - Add pagination (limit 50 per page)
> - Filter by date range
> - Sort by timestamp descending
> - Add error handling
```

### 2. Provide Constraints

```
> Add user authentication

Constraints:
- Use existing JWT library (PyJWT)
- Token expiry: 15 minutes
- Refresh token: 7 days
- Store refresh tokens in Redis (not database)
- Maximum 5 devices per user
```

### 3. Reference Existing Patterns

```
> Add caching to the account balance aggregation job

Follow the caching pattern used in:
- pipelines/users/aggregations.py (lines 45-60)
- Use DataFrame caching with persist(StorageLevel.MEMORY_AND_DISK)
- Invalidate on account updates
```

### 4. Specify Output Format

```
> Analyze Spark job performance

Return results as:

## Slow Stages
| Stage | Avg Time | Tasks | Shuffle Read |
|-------|----------|-------|--------------|

## Optimization Opportunities
- Job: payment_processing, Issue: Data skew on account_id
- Job: balance_aggregation, Issue: Small file problem

## Recommendations
1. [Specific action with code]
2. [Specific action with code]
```

### 5. Use Progressive Disclosure

Start simple, then add detail:

```
Phase 1:
> Explain how payment data processing works in this codebase

Phase 2:
> Now explain the error handling for failed payment records

Phase 3:
> Show me how to add a new payment method (ACH transfers)
```

### 6. Banking-Specific Tips

**Financial Calculations:**
```
> Calculate loan payment

IMPORTANT:
- Use integers (cents) not floats
- Round to nearest cent (banker's rounding)
- Show calculation steps for audit
- Validate against maximum/minimum values
```

**Security:**
```
> Generate password reset token

Security requirements:
- Cryptographically secure random (secrets.token_urlsafe)
- 32 bytes minimum
- One-time use only
- Expires in 1 hour
- Invalidate previous tokens
- Log generation and usage
```

**Compliance:**
```
> Implement user data deletion (GDPR)

Compliance requirements:
- Hard delete from primary database
- Purge from backups (mark for deletion)
- Clear from all caches
- Notify downstream systems
- Generate deletion audit report
- Retain audit logs (7 years)
```

---

## Common Mistakes to Avoid

### Mistake 1: Too Vague

❌ **Bad:**
```
> Check for bugs
```

✅ **Good:**
```
> Scan pipelines/payments/ for:
> - Off-by-one errors in loops
> - Null/None pointer exceptions
> - Race conditions in DataFrame operations
> - Resource leaks
```

### Mistake 2: Missing Context

❌ **Bad:**
```
> Add validation
```

✅ **Good:**
```
> Add validation to payment_processing pipeline
> Using Pydantic schema (like schemas/user_schema.py)
> Validate: account_id (UUID), amount (positive int), description (max 200 chars)
```

### Mistake 3: Unrealistic Expectations

❌ **Bad:**
```
> Rewrite the entire payment system to use streaming architecture
```

✅ **Good:**
```
> Identify which components in pipelines/payments/ could be migrated to streaming
> For each component, analyze:
> - Current batch dependencies
> - Stream processing boundaries
> - Data statefulness
> - Migration complexity
```

### Mistake 4: No Success Criteria

❌ **Bad:**
```
> Optimize the pipeline
```

✅ **Good:**
```
> Optimize payment_processing pipeline to:
> - Processing time < 5 minutes for 1M records
> - Support concurrent job execution
> - Reduce shuffle operations from N+1 to single shuffle
> - Measure improvement with benchmarks
```

---

## Summary

In this section, you learned:

### Fundamentals
- Basic prompt structure (Task + Context + Format)
- Being clear and direct
- Providing context
- Using roles via CLAUDE.md

### Intermediate Techniques
- Separating data from instructions
- Formatting output
- Chain of thought reasoning
- Few-shot learning with examples

### Advanced Techniques
- Avoiding hallucinations
- Verification prompts
- Multi-step complex prompts
- Iterative refinement

### Banking Applications
- Security review prompts
- Compliance checking
- Code generation with banking context
- Debugging with domain knowledge

### Prompt Templates
- Ready-to-use templates for common tasks
- Banking-specific adaptations
- Best practices and common mistakes

---

## Next Steps

1. **Practice**: Try the examples with your own codebase
2. **Experiment**: Create custom prompts for your team's needs
3. **Learn More**: Complete [Anthropic's Interactive Tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial)
4. **Share**: Create team prompt library in `.claude/commands/`

---

## Additional Resources

- **Anthropic Prompt Engineering Tutorial**: https://github.com/anthropics/prompt-eng-interactive-tutorial
- **Prompt Engineering Guide**: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering
- **Claude Prompt Library**: https://docs.anthropic.com/en/prompt-library/library
- **Best Practices**: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

---
