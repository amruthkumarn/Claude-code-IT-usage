# Phase 1.1: Introduction - Getting Started & Basics

**Learning Objectives:**
- Understand what Claude Code is and its capabilities
- Learn how Claude Code benefits Banking IT Data Chapter teams
- Complete your first Claude Code session
- Understand basic workflows for data engineering tasks

**Time Commitment:** 2-3 hours

**Prerequisites:** None (this is where you start!)

---

## Table of Contents
1. [What is Claude Code?](#what-is-claude-code)
2. [Why Use Claude Code in Banking IT Data Engineering?](#why-use-claude-code-in-banking-it-data-engineering)
3. [Key Features for Data Engineers](#key-features-for-data-engineers)
4. [Your First Session](#your-first-session)
5. [Common Data Engineering Workflows](#common-data-engineering-workflows)
6. [What You'll Learn in Phase 1](#what-youll-learn-in-phase-1)
7. [Success Criteria](#success-criteria)

---

## What is Claude Code?

Claude Code is Anthropic's official agentic coding tool that brings AI-powered assistance directly to your terminal. It's designed specifically to help data engineers with:

- **Building Pipelines**: Describe your data transformation requirements, and Claude will generate PySpark code
- **Debugging**: Analyze PySpark jobs, identify performance bottlenecks, and implement fixes
- **Code Navigation**: Understand complex data pipeline architectures
- **Task Automation**: Handle repetitive tasks like generating test cases, fixing linting issues, and creating documentation

### How It Works

Claude Code operates in your terminal and can:
- **Read** your codebase (PySpark pipelines, configuration files, schemas)
- **Edit** files directly (with your approval)
- **Execute** commands (`pytest`, `spark-submit`, `ruff`, `black`)
- **Search** through code and documentation
- **Integrate** with external tools via MCP (databases, Jira, monitoring systems)

🏦 **Banking IT Policy**: Git operations (commit, push, pull) must be executed **manually by developers**. Claude can draft commit messages and PR descriptions, but YOU execute the git commands.

### Architecture

```
┌─────────────────────────────────────────────┐
│           Your Terminal                      │
│  ┌────────────────────────────────────┐    │
│  │     Claude Code CLI (claude)       │    │
│  └─────────────┬──────────────────────┘    │
│                │                              │
│  ┌─────────────▼──────────────────────┐    │
│  │   Anthropic Claude AI (Sonnet)     │    │
│  └─────────────┬──────────────────────┘    │
│                │                              │
│  ┌─────────────▼──────────────────────┐    │
│  │     Your PySpark Codebase          │    │
│  │  • Read pipelines/*.py              │    │
│  │  • Edit transformation logic        │    │
│  │  • Run pytest, spark-submit         │    │
│  │  • Draft git messages (manual)      │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Model:** Sonnet 4.5 (currently available in AWS Bedrock for banking IT use)

---

## Why Use Claude Code in Banking IT Data Engineering?

### For Individual Data Engineers
- **Faster Development**: Generate PySpark boilerplate code 10x faster
- **Code Quality**: Get instant reviews for performance, security, and compliance
- **Learning**: Understand complex PySpark optimizations and best practices
- **Documentation**: Auto-generate data pipeline documentation and DDTs

### For Data Chapter Teams
- **Consistency**: Enforce PySpark coding standards across all pipelines
- **Knowledge Sharing**: Embed team conventions in CLAUDE.md files
- **Onboarding**: New engineers understand banking data pipelines faster
- **Compliance**: Automate PCI-DSS, SOX, GDPR compliance checks

### Banking-Specific Benefits
- **Security First**: Read-only by default, explicit approval required for edits
- **Audit Trail**: All actions logged and monitored (required for SOX compliance)
- **Regulatory Compliance**: Built-in compliance checks for sensitive data
- **Code Review**: Automated data quality and security scans before human review
- **Legacy Modernization**: Understand and refactor legacy banking ETL jobs

**Real Impact:**
- **Development Time**: 40-60% reduction in coding time
- **Bug Detection**: 70% of data quality issues caught before deployment
- **Onboarding**: 50% faster ramp-up for new team members

---

## Key Features for Data Engineers

### 1. Interactive REPL for Data Pipelines
Work conversationally with Claude in your terminal:

```bash
> Generate a PySpark pipeline to read customer transactions from S3,
> validate data quality, mask PII, and write to Delta Lake Bronze layer

> Optimize this aggregation query - it's taking 2 hours to run

> Add pytest test cases for the payment validation logic
```

### 2. Direct File Access
Claude reads and edits your PySpark files (with approval):
- `pipelines/ingestion/*.py`
- `pipelines/transformations/*.py`
- `tests/*.py`
- `schemas/*.py`

### 3. Command Execution
Run data engineering commands:
- `pytest tests/` - Run unit tests
- `spark-submit pipelines/main.py` - Submit Spark jobs
- `ruff check pipelines/` - Lint PySpark code
- `black pipelines/` - Format code

### 4. Plan Mode (Read-Only Exploration)
Explore pipelines safely without making changes:
```bash
claude --permission-mode plan
> Analyze the payment processing pipeline and suggest optimizations
```

### 5. Custom Slash Commands
Create banking-specific commands:
- `/data-quality-check` - Run DQ validation
- `/compliance-scan` - Check PCI-DSS/SOX/GDPR compliance
- `/generate-tests` - Generate pytest test cases
- `/optimize-spark` - Suggest performance improvements

### 6. Custom Agents
Specialized AI agents for data engineering:
- `@data-quality-auditor` - Data quality validation
- `@pyspark-optimizer` - Performance optimization
- `@compliance-checker` - Regulatory compliance checks
- `@test-generator` - Test case generation

### 7. MCP Integrations
Connect to banking systems:
- PostgreSQL (read-only access to production metadata)
- Jira (automated ticket creation)
- Databricks (job submission and monitoring)
- Datadog (performance metrics)

---

## Your First Session

Let's complete your first Claude Code session step-by-step.

### Prerequisites Check

Before starting, verify you have:
- ✅ Python 3.9+ installed
- ✅ Claude Code installed (we'll do this in Phase 1.2)
- ✅ Access to a PySpark project directory

### Step 1: Navigate to Your Project

```bash
# Navigate to your PySpark project
cd ~/projects/payment-processing-pipeline

# Verify you're in the right place
pwd
ls
```

You should see:
```
pipelines/
tests/
schemas/
requirements.txt
pyproject.toml
```

### Step 2: Start Claude Code

```bash
# Start the interactive REPL
claude
```

You'll see:
```
Welcome to Claude Code!
You're in: /Users/yourname/projects/payment-processing-pipeline

Type /help for available commands
Press ? for keyboard shortcuts

>
```

### Step 3: Ask Your First Question

```
> What is this project about?
```

Claude will:
1. Read `README.md`, `pyproject.toml`
2. Scan the `pipelines/` directory structure
3. Provide a summary of the data pipeline

Example response:
```
This is a payment processing data pipeline built with PySpark.

Structure:
- pipelines/ingestion/s3_reader.py - Reads payment data from S3
- pipelines/transformations/validation.py - Validates transactions
- pipelines/transformations/aggregation.py - Aggregates by merchant
- pipelines/output/delta_writer.py - Writes to Delta Lake

Tech Stack: PySpark 3.4, Delta Lake, pytest
```

### Step 4: Explore the Codebase

```
> Show me the payment validation logic
```

Claude will read `pipelines/transformations/validation.py` and explain it.

### Step 5: Ask for Help

```
> How do I add a new validation rule to check if transaction amount is within daily limits?
```

Claude will:
1. Read the current validation code
2. Suggest where to add the new rule
3. Generate the PySpark code
4. Ask for your approval before editing

### Step 6: Generate Code (with Approval)

```
> Add the daily limit validation rule
```

Claude will show you the changes and ask:
```
I'll add this to pipelines/transformations/validation.py:

[Shows diff]

Approve? (yes/no)
```

Type `yes` to approve, or `no` to reject.

### Step 7: Exit

```
Ctrl+D
```

or

```
> /exit
```

**Congratulations!** You've completed your first Claude Code session.

---

## Common Data Engineering Workflows

### Workflow 1: Understand Existing Pipeline

**Scenario**: You've joined the team and need to understand the customer aggregation pipeline.

```bash
claude --permission-mode plan

> Explain the customer aggregation pipeline in pipelines/customer_aggregation.py
> What data sources does it use?
> What transformations are applied?
> Where is the output written?
> What are the potential performance bottlenecks?
```

**Benefit**: Understand complex pipelines in minutes instead of hours.

---

### Workflow 2: Generate New Pipeline from Requirements

**Scenario**: Business wants a new daily payment summary pipeline.

```bash
claude

> Generate a PySpark pipeline with these requirements:
> - Read payment transactions from S3 (s3://banking-data/payments/raw/)
> - Schema: transaction_id, account_id, amount, merchant_id, timestamp
> - Filter out failed transactions (status != 'SUCCESS')
> - Aggregate by merchant_id and date
> - Calculate: total_amount, transaction_count, avg_amount
> - Write to Delta Lake (s3://banking-data/payments/aggregated/)
> - Include data quality checks: no nulls, amount > 0
> - Add PII masking for account_id
> - Include error handling and audit logging
```

Claude will:
1. Generate the full PySpark pipeline
2. Include data quality validations
3. Add PII masking (banking compliance)
4. Include audit logging
5. Ask for your review before creating files

**Time Saved**: 2 hours → 15 minutes

---

### Workflow 3: Fix a Bug

**Scenario**: Aggregation is producing incorrect totals.

```bash
claude

> The payment aggregation in pipelines/aggregation.py is producing incorrect daily totals.
> Debug and fix the issue.
```

Claude will:
1. Read the aggregation code
2. Identify potential issues (e.g., duplicate counting, wrong groupBy)
3. Suggest fixes
4. Generate a test case to reproduce the bug
5. Implement the fix (with your approval)

---

### Workflow 4: Optimize Slow Pipeline

**Scenario**: Customer transformation job takes 3 hours.

```bash
claude

> Analyze pipelines/customer_transformation.py for performance issues.
> The job processes 500M records and takes 3 hours.
> Suggest optimizations.
```

Claude will:
1. Read the PySpark code
2. Identify optimization opportunities:
   - Add partitioning
   - Use broadcast joins for small dimensions
   - Cache intermediate DataFrames
   - Optimize shuffle operations
3. Implement optimizations (with approval)
4. Estimate performance improvement

**Result**: 3 hours → 45 minutes (4x faster)

---

### Workflow 5: Generate Test Cases

**Scenario**: You need pytest tests for validation logic.

```bash
claude

> Generate pytest test cases for pipelines/validation.py
> Include edge cases: null values, negative amounts, invalid dates
> Use SparkSession fixtures
```

Claude will:
1. Read the validation code
2. Generate comprehensive pytest tests
3. Include fixtures for test data
4. Cover edge cases
5. Aim for 80%+ code coverage

---

### Workflow 6: Create Documentation

**Scenario**: Need technical documentation for the payment pipeline.

```bash
claude

> Generate documentation for the payment processing pipeline.
> Include: architecture, data flow, schema, transformations, error handling
> Format as Markdown
```

Claude will create comprehensive documentation including:
- Pipeline architecture diagram
- Data flow (source → bronze → silver → gold)
- Schema definitions (StructType)
- Transformation logic
- Data quality rules
- Error handling approach

---

## What You'll Learn in Phase 1

**Phase 1: Onboarding** (2-3 weeks) will teach you:

### 1.1 Introduction - Getting Started (This Section)
- What Claude Code is
- Why it's valuable for data engineering
- Your first session

### 1.2 Installation
- Install Claude Code (pip method)
- Configure for banking IT environment
- Authenticate and verify
- Troubleshoot common issues

### 1.3 Core Concepts & Features
- How Claude Code works (architecture)
- Security & permissions model
- Memory & context management
- CLI reference & commands
- **Project configuration** (settings.json)
- **Slash commands** (custom automation)
- **Agents & sub-agents** (specialized AI assistants)
- **Hooks & automation** (compliance enforcement)
- **MCP integration** (connect to banking systems)
- **Git integration** (manual operations policy)
- **Standards & best practices** (code quality)
- **Templates library** (reusable patterns)

### 1.4 Prompt Engineering for Data Engineering
- Write effective prompts for PySpark code generation
- Avoid hallucinations (critical for banking)
- Complex multi-step prompts (BRD → DDT → Code)
- Banking-specific prompt library

### 1.5 Assessment
- Hands-on exercises (10 practical tasks)
- Knowledge quiz (30 questions)
- Practical project (build a pipeline end-to-end)
- Self-evaluation

**By the end of Phase 1**, you will:
- ✅ Be proficient with Claude Code
- ✅ Understand all features and configuration options
- ✅ Write effective prompts for data engineering
- ✅ Configure security and compliance settings
- ✅ Be ready to build production pipelines in Phase 2

---

## Success Criteria

You're ready to move to Phase 1.2 (Installation) when you can:

✅ **Explain** what Claude Code is and how it works
✅ **Describe** at least 5 benefits for banking data engineering
✅ **List** the key features (REPL, file access, commands, plan mode, slash commands, agents, MCP)
✅ **Complete** a basic Claude Code session (start, ask question, exit)
✅ **Understand** the Banking IT git policy (manual operations only)

---

## Quick Reference

**Essential Commands:**
```bash
claude                    # Start Claude Code
claude --permission-mode plan  # Read-only mode
/help                     # Show available commands
/exit                     # Exit Claude Code
Ctrl+D                    # Alternative exit
```

**First Session Checklist:**
- [ ] Navigate to project directory
- [ ] Start Claude Code (`claude`)
- [ ] Ask "What is this project about?"
- [ ] Explore the codebase
- [ ] Ask a simple question
- [ ] Exit cleanly

---

## Next Steps

**Ready to install Claude Code?**

👉 **[Continue to Phase 1.2: Installation](./02-installation.md)**

**Need help?**
- Type `/help` in Claude Code
- Check [Troubleshooting FAQ](../reference/troubleshooting-faq.md)
- Ask your team lead

---

## Summary

In this section, you learned:
- ✅ What Claude Code is and its architecture
- ✅ Why it's valuable for Banking IT Data Chapter
- ✅ Key features for data engineers (REPL, commands, plan mode, slash commands, agents, MCP)
- ✅ How to complete your first session
- ✅ Common data engineering workflows
- ✅ What you'll learn in Phase 1 (onboarding)

**Time Spent**: ~30 minutes reading + 30 minutes first session = 1 hour

**Next**: Install Claude Code and configure it for banking IT environment → [Phase 1.2: Installation](./02-installation.md)
