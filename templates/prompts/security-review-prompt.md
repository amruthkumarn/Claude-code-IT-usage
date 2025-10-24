# Security Review Prompt Template

Use this template for comprehensive security audits of data pipelines and Python code.

## Prompt

```
Perform comprehensive security review of [FILE_OR_DIRECTORY]

Check for:

## Data Access & Authorization
- All data sources require authentication?
- Proper role-based access control for sensitive data?
- Service principal/IAM validation correct?
- Spark session security configured?

## Data Validation & Injection Prevention
- Spark SQL injection prevention (dynamic SQL, string formatting)?
- Schema validation on DataFrame inputs?
- Data type validation and constraints?
- PII detection and masking applied?

## Data Protection
- Sensitive data encrypted at rest?
- Sensitive data encrypted in transit?
- No hardcoded credentials?
- Secrets managed properly?

## Logging & Monitoring
- Data access events logged?
- PII excluded from logs and error messages?
- Pipeline failures don't leak sensitive data?
- Audit trail for data transformations?

## Banking-Specific Compliance
- PCI-DSS: No card data in logs or intermediate DataFrames?
- PCI-DSS: No CVV/CVV2 storage in any pipeline stage?
- PCI-DSS: Encryption at rest for cardholder data?
- SOX: Audit trail for financial data transformations?
- GDPR: Personal data properly masked/anonymized?
- GDPR: Data retention policies implemented?

For each issue found, provide:
- File and line number
- Issue description
- Severity (Critical/High/Medium/Low)
- Code example showing the issue
- Recommended fix with code

Prioritize critical security issues.
```

## Example Usage

```bash
> Perform comprehensive security review of pipelines/payment_processing/

[Claude will execute the security audit]
```

## Customize

Replace placeholders:
- `[FILE_OR_DIRECTORY]` - Target for review
- Add/remove specific checks based on your requirements
- Adjust severity levels for your organization
