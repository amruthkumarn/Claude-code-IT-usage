# Compliance Check Prompt Template

Use this template for regulatory compliance audits.

## Prompt

```
Audit [FILE_OR_DIRECTORY] for regulatory compliance.

Standards to check:

## PCI-DSS (Payment Card Industry)
- Cardholder data storage (encrypted?)
- CVV/CVV2 codes (must NEVER be stored)
- Encryption at rest and in transit
- Access controls to payment data
- Audit logging of access to cardholder data

## SOX (Sarbanes-Oxley)
- Financial transaction audit trails
- Change management controls
- Separation of duties in code
- Data integrity for financial records

## GDPR (General Data Protection Regulation)
- Personal data identification and classification
- Consent tracking mechanism
- Right to deletion implementation
- Data retention policies
- Cross-border data transfer handling

Generate report with:

### Compliance Status
For each requirement:
- ✅ Compliant: [Evidence/Code location]
- ❌ Non-compliant: [Issue description]
- ⚠️  Partially compliant: [What's missing]

### Non-Compliant Code Locations
- File: [path]
- Line: [number]
- Requirement: [Which regulation/requirement]
- Issue: [Description]
- Risk Level: [Critical/High/Medium/Low]

### Remediation Steps
1. [Action item with code example]
2. [Action item with code example]

### Priority Ranking
1. Critical (PCI-DSS violations)
2. High (SOX audit trail gaps)
3. Medium (GDPR improvements)
```

## Example Usage

```bash
> Audit src/transactions/ for regulatory compliance.

[Claude will execute the compliance check]
```

## Banking Customization

Add organization-specific requirements:
```
## Additional Requirements
- Internal Policy XYZ
- Regional Regulation ABC
- Industry Standard DEF
```
