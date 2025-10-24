# Phase 1.4: Prompt Engineering for Data Engineering

**Overview:** Master the art of effective prompts to maximize Claude Code's capabilities for banking data engineering tasks.

**Total Time Commitment:** 4-5 hours

**Prerequisites:** Phase 1.1-1.3 completed

---

## What You'll Learn

This phase teaches you prompt engineering techniques specifically adapted for banking IT data engineering workflows. Based on Anthropic's official interactive tutorial, each subsection includes:

- **Core Concepts**: Fundamental prompt engineering principles
- **PySpark Examples**: Real banking data pipeline scenarios
- **Hands-On Exercises**: Practice with actual data engineering tasks
- **Best Practices**: Banking compliance and security considerations

---

## Tutorial Structure

### 📚 Core Chapters (9 Subsections)

**Foundations (Subsections 1-3):**
1. **[Tutorial How-To](./04-01-tutorial-how-to.md)** *(10 min)*
   - How to use this tutorial
   - Setting up your environment
   - Interactive exercises format

2. **[Basic Prompt Structure](./04-02-basic-prompt-structure.md)** *(30 min)*
   - Task + Context + Format formula
   - PySpark pipeline generation examples
   - Banking data validation prompts

3. **[Being Clear and Direct](./04-03-being-clear-direct.md)** *(30 min)*
   - Specific vs vague prompts for data engineering
   - Transaction processing examples
   - ETL pipeline instructions

**Context & Roles (Subsections 4-5):**

4. **[Assigning Roles (Role Prompting)](./04-04-assigning-roles.md)** *(30 min)*
   - Using CLAUDE.md for data engineering context
   - Banking domain expert persona
   - Compliance-aware code generation

5. **[Separating Data and Instructions](./04-05-separating-data-instructions.md)** *(30 min)*
   - Using XML tags for schema definitions
   - Markdown code blocks for sample data
   - Delimiters for SQL queries

**Output Control (Subsections 6-7):**

6. **[Formatting Output](./04-06-formatting-output.md)** *(30 min)*
   - JSON schemas for validation rules
   - Markdown tables for data lineage
   - Structured reports for compliance audits

7. **[Precognition (Thinking Step-by-Step)](./04-07-precognition-step-by-step.md)** *(45 min)*
   - Chain of thought for complex transformations
   - Debugging PySpark performance issues
   - Data quality validation reasoning

**Advanced Techniques (Subsections 8-9):**

8. **[Using Examples (Few-Shot Prompting)](./04-08-using-examples.md)** *(45 min)*
   - Test case generation patterns
   - Schema validation examples
   - Error handling templates

9. **[Avoiding Hallucinations](./04-09-avoiding-hallucinations.md)** *(30 min)*
   - Grounding in actual codebase
   - Preventing invented APIs
   - Verification strategies for banking code

### 📖 Appendix (Subsections 10-13)

**Advanced Workflows:**

10. **[Complex Prompts from Scratch](./04-10-complex-prompts.md)** *(45 min)*
    - Multi-phase pipeline development
    - End-to-end ETL workflows
    - Compliance + performance + security

11. **[Chaining Prompts](./04-11-chaining-prompts.md)** *(30 min)*
    - Sequential prompt workflows
    - Pipeline → Tests → Docs → Deploy
    - Iterative refinement patterns

12. **[Tool Use](./04-12-tool-use.md)** *(30 min)*
    - Leveraging Claude Code tools effectively
    - Read → Analyze → Edit workflows
    - Using agents for specialized tasks

13. **[Search & Retrieval](./04-13-search-retrieval.md)** *(30 min)*
    - Finding code patterns in large codebases
    - Schema discovery across tables
    - Dependency analysis

---

## Learning Path

### 🎯 Recommended Order

**For Beginners (Complete Tutorial):**
```
Start → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → End
Optional: 11, 12, 13 (Appendix)
```

**For Experienced Developers (Fast Track):**
```
Start → 01 → 04 → 07 → 08 → 09 → 10 → End
```

**For Banking Compliance Focus:**
```
Start → 02 → 04 → 06 → 09 → 10 → End
```

---

## What's Different from Anthropic's Tutorial?

### Adaptations for Banking IT Data Engineering

| Anthropic Tutorial | This Tutorial |
|-------------------|---------------|
| General coding examples | PySpark data pipeline examples |
| Web development focus | Banking transaction processing |
| Generic AI tasks | Compliance, security, data quality |
| JavaScript/Python mix | Python + PySpark only |
| Simple demonstrations | Production-grade patterns |

### Banking-Specific Additions

✅ **Compliance Focus:**
- PCI-DSS prompts for payment data
- SOX audit trail generation
- GDPR data handling examples

✅ **Data Engineering Patterns:**
- Schema validation with StructType
- DataFrame transformations
- Delta Lake operations
- Data quality checks

✅ **Security Prompts:**
- Secrets detection
- PII masking
- SQL injection prevention
- Encryption patterns

✅ **Production Scenarios:**
- Error handling in pipelines
- Performance optimization
- Monitoring and alerting
- Incident response

---

## Hands-On Practice Environment

### Prerequisites

Before starting the tutorial:

```bash
# 1. Ensure Claude Code is installed
claude --version

# 2. Create practice project
mkdir prompt-engineering-practice
cd prompt-engineering-practice

# 3. Initialize Claude Code configuration
mkdir -p .claude/{commands,hooks}

# 4. Create sample data engineering project structure
mkdir -p {pipelines,schemas,tests,config}
```

### Sample Dataset

Each subsection includes sample banking data:

```python
# Sample transactions for practice
sample_transactions = [
    {"txn_id": "TXN001", "account_id": "ACC123", "amount": 100.50, "currency": "USD"},
    {"txn_id": "TXN002", "account_id": "ACC456", "amount": 250.00, "currency": "USD"},
    {"txn_id": "TXN003", "account_id": "ACC123", "amount": -50.25, "currency": "USD"}
]
```

### Interactive Format

Each subsection follows this structure:

1. **Concept Introduction** - Theory and principles
2. **Data Engineering Example** - Banking scenario
3. **Bad Prompt** - What NOT to do
4. **Good Prompt** - Effective approach
5. **Claude's Response** - Expected output
6. **Practice Exercise** - Try it yourself
7. **Solution** - Detailed explanation

---

## Success Criteria

By the end of Phase 1.4, you should be able to:

### Core Skills
- ✅ Structure effective prompts for data engineering tasks
- ✅ Generate PySpark pipelines with clear instructions
- ✅ Create compliance-aware code (PCI-DSS, SOX, GDPR)
- ✅ Debug complex data pipeline issues with Claude
- ✅ Generate comprehensive test suites for data pipelines

### Advanced Skills
- ✅ Design multi-step prompt workflows
- ✅ Chain prompts for end-to-end development
- ✅ Use role prompting for specialized tasks
- ✅ Avoid hallucinations in banking code generation
- ✅ Format outputs for specific use cases (JSON, Markdown, reports)

### Banking IT Skills
- ✅ Generate PCI-DSS compliant payment processing code
- ✅ Create SOX audit trail implementations
- ✅ Implement GDPR data handling patterns
- ✅ Optimize PySpark performance with targeted prompts
- ✅ Generate data quality validation logic

---

## Quick Reference: Prompt Templates

### Pipeline Generation
```
> Generate a PySpark pipeline for [BUSINESS_LOGIC]

Input schema: [StructType definition]
Transformations: [Step-by-step logic]
Output schema: [StructType definition]
Data quality checks: [Validation rules]
Compliance: [PCI-DSS/SOX/GDPR requirements]
```

### Debugging
```
> Debug [PIPELINE_NAME] which is [SYMPTOM]

Error logs: [Paste error messages]
Expected behavior: [What should happen]
Recent changes: [Commits or modifications]
Investigate: [Specific areas to check]
```

### Code Review
```
> Review [FILE_PATH] for:
- Security vulnerabilities
- PySpark best practices
- Banking compliance (PCI-DSS, SOX, GDPR)
- Performance optimization opportunities
- Data quality checks

Provide file:line references and code examples.
```

### Test Generation
```
> Generate pytest tests for [FUNCTION/CLASS]

Test cases:
- Happy path with sample DataFrames
- Edge cases (nulls, empty datasets, schema mismatches)
- Error conditions (network failures, invalid data)
- Banking scenarios (overdraft, duplicate transactions, compliance)

Use pyspark.testing.utils.assertDataFrameEqual
```

---

## Resources

### Anthropic Official Resources
- **Interactive Tutorial**: https://github.com/anthropics/prompt-eng-interactive-tutorial
- **Prompt Engineering Guide**: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering
- **Prompt Library**: https://docs.anthropic.com/en/prompt-library/library

### Banking Data Engineering Resources
- **PySpark Documentation**: https://spark.apache.org/docs/latest/api/python/
- **Delta Lake Guide**: https://docs.delta.io/latest/index.html
- **PCI-DSS Standards**: https://www.pcisecuritystandards.org/
- **Great Expectations**: https://docs.greatexpectations.io/ (Data quality)

---

## Getting Help

### If You Get Stuck

1. **Review Prerequisites**: Ensure Phase 1.1-1.3 completed
2. **Check Examples**: Each subsection has working examples
3. **Try Plan Mode**: Use `claude --permission-mode plan` for exploration
4. **Ask Claude**: Use the tutorial prompts as templates
5. **Practice More**: Repetition builds prompt engineering skills

### Common Issues

**Issue**: Claude generates code that doesn't match my project structure
**Solution**: Use CLAUDE.md to define your project context (see subsection 04-04)

**Issue**: Prompts are too vague, outputs vary
**Solution**: Follow Task + Context + Format structure (see subsection 04-02)

**Issue**: Claude invents APIs or functions that don't exist
**Solution**: Ground prompts in actual codebase (see subsection 04-09)

---

## Next Steps After Phase 1.4

👉 **[Continue to Phase 1.5: Assessment](./05-assessment.md)** - Test your knowledge

**Or explore:**
- Create custom slash commands using your new prompt skills
- Build a team prompt library in `.claude/commands/`
- Practice with real banking data pipeline scenarios
- Share prompt patterns with your team

---

## Subsection Index

### Core Chapters
1. [Tutorial How-To](./04-01-tutorial-how-to.md)
2. [Basic Prompt Structure](./04-02-basic-prompt-structure.md)
3. [Being Clear and Direct](./04-03-being-clear-direct.md)
4. [Assigning Roles (Role Prompting)](./04-04-assigning-roles.md)
5. [Separating Data and Instructions](./04-05-separating-data-instructions.md)
6. [Formatting Output](./04-06-formatting-output.md)
7. [Precognition (Thinking Step-by-Step)](./04-07-precognition-step-by-step.md)
8. [Using Examples (Few-Shot Prompting)](./04-08-using-examples.md)
9. [Avoiding Hallucinations](./04-09-avoiding-hallucinations.md)

### Appendix
10. [Complex Prompts from Scratch](./04-10-complex-prompts.md)
11. [Chaining Prompts](./04-11-chaining-prompts.md)
12. [Tool Use](./04-12-tool-use.md)
13. [Search & Retrieval](./04-13-search-retrieval.md)

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Prompt Engineering Interactive Tutorial
