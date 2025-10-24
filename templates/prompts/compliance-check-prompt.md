# Compliance Check Prompt Template

Use this template for regulatory compliance audits of data pipelines.

## Prompt

```
Audit [FILE_OR_DIRECTORY] for regulatory compliance.

Standards to check:

## PCI-DSS (Payment Card Industry)
- Cardholder data storage in DataFrames/Delta tables (encrypted?)
- CVV/CVV2 codes (must NEVER be stored in any pipeline stage)
- Encryption at rest (Delta Lake encryption) and in transit (TLS)
- Access controls to payment data pipelines and storage
- Audit logging of access to cardholder data (data lineage tracking)

## SOX (Sarbanes-Oxley)
- Financial transaction data audit trails (data lineage)
- Change management controls for pipeline code
- Separation of duties in data access and transformations
- Data integrity validation for financial records (checksums, row counts)
- Immutable audit logs for data modifications

## GDPR (General Data Protection Regulation)
- Personal data identification and classification in pipelines
- Consent tracking in customer data tables
- Right to deletion implementation (data purge jobs)
- Data retention policies enforced in pipeline logic
- Cross-border data transfer handling (data residency checks)
- PII masking/anonymization in non-production environments

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
> Audit pipelines/transaction_processing/ for regulatory compliance.

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
