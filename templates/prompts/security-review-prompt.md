# Security Review Prompt Template

Use this template for comprehensive security audits of code.

## Prompt

```
Perform comprehensive security review of [FILE_OR_DIRECTORY]

Check for:

## Authentication & Authorization
- All endpoints require authentication?
- Proper role-based access control?
- JWT validation correct?
- Session management secure?

## Input Validation
- SQL injection prevention?
- XSS prevention?
- CSRF protection?
- Input sanitization?

## Data Protection
- Sensitive data encrypted at rest?
- Sensitive data encrypted in transit?
- No hardcoded credentials?
- Secrets managed properly?

## Logging & Monitoring
- Security events logged?
- PII excluded from logs?
- Error messages don't leak info?

## Banking-Specific
- PCI-DSS: No card data in logs?
- PCI-DSS: No CVV storage?
- SOX: Audit trail for financial transactions?
- GDPR: Personal data properly handled?

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
> Perform comprehensive security review of src/payments/

[Claude will execute the security audit]
```

## Customize

Replace placeholders:
- `[FILE_OR_DIRECTORY]` - Target for review
- Add/remove specific checks based on your requirements
- Adjust severity levels for your organization
