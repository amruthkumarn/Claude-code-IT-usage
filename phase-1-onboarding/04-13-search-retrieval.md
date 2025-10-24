# Phase 1.4.13: Search & Retrieval (Appendix)

**Learning Objectives:**
- Find code patterns in large codebases
- Discover schema and table dependencies
- Perform impact analysis for changes
- Master codebase navigation techniques

**Time Commitment:** 30 minutes

**Prerequisites:** Phase 1.4.1-1.4.12 completed

---

## ⚡ Quick Start (3 minutes)

**Goal:** See how effective search accelerates understanding large codebases.

```bash
claude
```

**In the Claude REPL:**

**❌ INEFFICIENT (No Search Strategy):**
```
> Tell me about our validation logic
```

**✅ EFFICIENT (Strategic Search):**
```
> Find and analyze our validation logic

Search strategy:
1. Grep for 'def validate_' to find all validation functions
2. Glob for '*validator*.py' to find validation modules
3. Read top 3 most important validators
4. Summarize the validation patterns we use
```

**Exit Claude:**
```bash
Ctrl+D
```

**Key Insight:** Strategic search finds exactly what you need in large codebases!

---

## Table of Contents
1. [Search Strategies](#search-strategies)
2. [Finding Code Patterns](#finding-code-patterns)
3. [Impact Analysis](#impact-analysis)
4. [Banking Data Engineering Examples](#banking-data-engineering-examples)
5. [Practice Exercises](#practice-exercises)
6. [Summary](#summary)

---

## Search Strategies

### Strategy 1: Top-Down Discovery

**Use When:** You know the feature, need to find implementation

**Pattern:**
```
1. Glob for files by name pattern
2. Grep for key functions/classes
3. Read relevant files
4. Summarize findings
```

**Example:**
```
> Find all fraud detection logic

Step 1: Find files
Glob: **/*fraud*.py

Step 2: Find key functions
Grep: 'def.*fraud|class.*Fraud' in files from step 1

Step 3: Understand implementation
Read top 3 fraud-related files

Step 4: Summary
List all fraud detection rules and their locations
```

---

### Strategy 2: Bottom-Up Discovery

**Use When:** You have a specific function, need to understand its usage

**Pattern:**
```
1. Grep for function calls
2. Read calling code
3. Trace data flow
4. Map dependencies
```

**Example:**
```
> Find all usages of validate_transaction_amount()

Step 1: Find calls
Grep: 'validate_transaction_amount' across all Python files

Step 2: Analyze each call site
For each file with matches:
  Read file
  Understand context of call

Step 3: Map usage
Create list of where and how function is used

Step 4: Impact assessment
If we change this function, what breaks?
```

---

### Strategy 3: Schema Discovery

**Use When:** Need to understand data structures and dependencies

**Pattern:**
```
1. Grep for schema definitions (StructType, CREATE TABLE)
2. Grep for schema usage (read_table, write_table)
3. Map data lineage
4. Identify dependencies
```

**Example:**
```
> Map the transactions table lifecycle

Step 1: Find schema definition
Grep: 'transaction.*StructType|CREATE TABLE.*transaction'

Step 2: Find reads
Grep: 'spark.read.*transaction|read_table.*transaction'

Step 3: Find writes
Grep: 'write.*transaction|insertInto.*transaction'

Step 4: Build lineage map
source → transformations → destination
```

---

## Finding Code Patterns

### Pattern 1: Find All Implementations of Interface

**Prompt:**
```
> Find all validator implementations

Search workflow:

1. Find validator interface/base class
Grep: 'class.*Validator|def validate_'

2. Find all implementations
Glob: **/*validator*.py
List all files

3. Extract pattern
Read 3 representative validators
Identify common structure

4. List all validators
File | Function | Purpose | Validation Rules
```

---

### Pattern 2: Find Error Handling Patterns

**Prompt:**
```
> Discover error handling patterns in our ETL pipelines

Search strategy:

1. Find all exception handling
Grep: 'try:|except' in pipelines/etl/**/*.py
Count occurrences per file

2. Analyze top 3 files with most try/except
Read files with highest exception handling

3. Extract patterns
- What exceptions are caught?
- How are they logged?
- What retry logic exists?
- How are failures propagated?

4. Document standard pattern
Create template for team based on findings
```

---

### Pattern 3: Find Configuration Usage

**Prompt:**
```
> Find all Spark configuration settings across codebase

Search workflow:

1. Find config.set patterns
Grep: 'spark.conf.set|spark.config|sparkConf.set'

2. Categorize configs
Group by:
- Performance tuning
- Security settings
- Execution settings
- Storage settings

3. Find hardcoded values
Identify configs that should be in config files

4. Recommend consolidation
Suggest moving to central config management
```

---

## Impact Analysis

### Impact Analysis 1: Schema Change Impact

**Prompt:**
```
> Analyze impact of adding merchant_category field to transactions table

Impact analysis workflow:

Step 1: Find schema definition
Grep: 'transaction.*schema|StructType.*txn_id'
Read: schemas/transaction.py

Step 2: Find all reads of transactions table
Grep: 'spark.read.*transaction|spark.table.*transaction'
List all files reading from transactions

Step 3: Find all writes to transactions table
Grep: 'write.*transaction|insertInto.*transaction'
List all files writing to transactions

Step 4: Analyze each file
For each file from steps 2 and 3:
  Read file
  Determine if it needs updates for new field
  Assess: Breaking change? Optional enhancement? No impact?

Step 5: Impact report
Create table:
File | Impact Level | Change Required | Effort | Priority
```

---

### Impact Analysis 2: Function Refactoring Impact

**Prompt:**
```
> Analyze impact of changing validate_amount() signature

Impact analysis:

Step 1: Find function definition
Grep: 'def validate_amount'
Read: pipelines/validators/amount_validator.py

Current signature: validate_amount(df: DataFrame) -> DataFrame
Proposed: validate_amount(df: DataFrame, min_amount: Decimal, max_amount: Decimal) -> Tuple[DataFrame, DataFrame]

Step 2: Find all call sites
Grep: 'validate_amount\('
List all occurrences with file:line

Step 3: Analyze each caller
For each call site:
  Read surrounding code
  Determine current usage pattern
  Plan migration to new signature

Step 4: Dependency graph
Create call graph showing:
- Direct callers
- Indirect callers (functions calling direct callers)
- Test files affected

Step 5: Migration plan
Prioritized list of changes:
1. Update function signature (breaking change)
2. Update direct callers (N files)
3. Update tests (M test files)
4. Update documentation
```

---

## Banking Data Engineering Examples

### Example 1: Find All PCI-DSS Compliance Implementations

**Prompt:**
```
> Audit PCI-DSS compliance implementations across codebase

Comprehensive search:

Step 1: Find PCI-DSS mentions
Grep: 'PCI|pci|card.*mask|CVV|PAN' (case insensitive)
List all files with PCI-related code

Step 2: Find masking implementations
Grep: 'mask|redact|obfuscate' in files from step 1
Extract masking patterns

Step 3: Find audit logging
Grep: 'audit.*log|compliance.*log'
Identify audit trail implementations

Step 4: Find encryption usage
Grep: 'encrypt|cipher|AES'
List encryption implementations

Step 5: Compliance report
For each PCI-DSS requirement:
- Requirement ID
- Implementation files
- Coverage assessment (✅/⚠️/❌)
- Gaps identified
```

---

### Example 2: Map Data Lineage for Reporting

**Prompt:**
```
> Map complete data lineage for daily_transaction_report

Lineage mapping workflow:

Step 1: Find report generation code
Grep: 'daily_transaction_report'
Read: pipelines/reporting/daily_transaction_report.py

Step 2: Trace source tables
From report code, extract all:
- spark.read calls
- spark.table calls
Document: List of source tables

Step 3: For each source table, find its sources
Recursive search:
  For each table in sources:
    Grep: 'write.*{table_name}|insertInto.*{table_name}'
    Find which pipeline writes to it
    Repeat until reaching raw data sources

Step 4: Map transformations
For each pipeline in lineage:
  Read pipeline code
  Document transformations applied

Step 5: Generate lineage diagram
ASCII or Mermaid diagram:
Raw Data → Pipeline A → Intermediate Table → Pipeline B → Report

Step 6: Document lineage
Table: Source | Pipeline | Transformation | Destination | Owner | SLA
```

---

### Example 3: Find Performance Bottlenecks

**Prompt:**
```
> Find potential performance bottlenecks in transaction processing pipelines

Performance analysis search:

Step 1: Find shuffle operations
Grep: 'groupBy|join|repartition|sort' in pipelines/etl/**/*.py
List files with shuffle-heavy operations

Step 2: Find collect/take calls (danger!)
Grep: '\.collect\(|\.take\('
Flag any usage of collect() on large datasets

Step 3: Find missing optimizations
Grep for absence of patterns:
  Missing: 'broadcast' (should use broadcast joins)
  Missing: 'cache|persist' (might benefit from caching)
  Missing: 'partitionBy' (might need better partitioning)

Step 4: Analyze top 3 slowest patterns
Read files from step 1 with most shuffle operations
Identify optimization opportunities

Step 5: Recommendation report
File | Operation | Current Approach | Optimization | Est. Impact
```

---

### Example 4: Find Deprecated Pattern Usage

**Prompt:**
```
> Find all usage of deprecated RDD API (should use DataFrame API)

Deprecation audit:

Step 1: Find RDD usage
Grep: '\.rdd|sc\.parallelize|sc\.textFile|map\(lambda|reduceByKey'
List all files using RDD API

Step 2: Categorize RDD usage
For each file:
  Read context around RDD usage
  Determine: Can this be replaced with DataFrame API?

Step 3: Find complex RDD operations
Grep: 'reduceByKey|combineByKey|aggregateByKey'
These need careful migration

Step 4: Migration plan
For each RDD usage:
  Complexity: Low | Medium | High
  DataFrame equivalent
  Migration effort: Hours
  Priority: High | Med | Low

Step 5: Create migration guide
Document pattern replacements:
RDD Pattern → DataFrame Equivalent
Example code for each
```

---

## Practice Exercises

### 📝 Exercise: Impact Analysis for Breaking Change

**Scenario:** You need to change the transaction_schema to make merchant_id non-nullable.

**Your Task:** Write a prompt to analyze full impact.

**Include:**
- Find schema definition
- Find all usages
- Identify breaking changes
- Estimate effort

---

### ✅ Solution

**Effective Prompt:**
```
> Analyze impact of making merchant_id non-nullable in transaction_schema

Impact analysis workflow:

Step 1: Current state
Grep: 'StructField.*merchant_id' in schemas/
Read: schemas/transaction.py
Current: StructField("merchant_id", StringType(), nullable=True)
Proposed: StructField("merchant_id", StringType(), nullable=False)

Step 2: Find all table reads
Grep: 'spark.read.*transaction|spark.table.*transaction'
List: All pipelines reading transactions

Step 3: Find existing null handling
For each file from step 2:
  Grep: 'merchant_id.*isNull|isnull.*merchant_id|merchant_id.*fillna'
  Identify current null handling logic

Step 4: Data quality assessment
Grep: 'merchant_id' in data quality checks
Current null rate: Need to check actual data
Query: SELECT COUNT(*) WHERE merchant_id IS NULL

Step 5: Breaking change analysis
Files requiring updates:
- Pipelines assuming merchant_id can be null
- Validation logic that allows null
- Test data with null merchant_id
- Documentation mentioning optional merchant_id

Step 6: Migration plan
Pre-requisites:
1. Backfill null merchant_ids (or decide on default value)
2. Update validation to enforce non-null
3. Update all pipelines to handle as required field
4. Update tests
5. Update schema
6. Deploy in order: validation → pipelines → schema

Effort estimate: [X] hours
Risk level: HIGH (breaking change to core schema)
```

---

## Summary

### Search Strategies
- ✅ Top-down: Feature → Implementation
- ✅ Bottom-up: Function → Usages
- ✅ Schema: Definition → Lineage → Dependencies
- ✅ Pattern: Find → Analyze → Document

### Impact Analysis
- ✅ Schema changes: Find all readers/writers
- ✅ Function changes: Find all call sites
- ✅ Breaking changes: Map dependencies

### Banking Applications
- ✅ Compliance audits (PCI-DSS, SOX)
- ✅ Data lineage mapping
- ✅ Performance bottleneck discovery
- ✅ Deprecation tracking

---

## Key Takeaways

**Search Workflow Pattern:**
```
1. Define what you're looking for (pattern, function, schema)
2. Use Grep/Glob to find candidates
3. Read relevant files for context
4. Map relationships and dependencies
5. Document findings
6. Assess impact or provide recommendations
```

**Best Practices:**
- **Start broad** (Glob for files)
- **Narrow down** (Grep for specific patterns)
- **Deep dive** (Read key files)
- **Map connections** (dependencies, call graphs)
- **Document** (create reference for team)

**Common Search Patterns:**
- `Grep: 'def {function_name}'` - Find function definitions
- `Grep: '{function_name}\('` - Find function calls
- `Grep: 'class {ClassName}'` - Find class definitions
- `Glob: **/*{pattern}*.py` - Find files by name
- `Grep: 'StructType|CREATE TABLE'` - Find schemas

---

## Next Steps

👉 **[Return to Phase 1.4 Index](./04-00-prompt-engineering-index.md)** - Review all techniques

**Or continue to:**
- **[Phase 1.5: Assessment](./05-assessment.md)** - Test your Phase 1 knowledge

**Practice:**
1. ✅ Use search to map a feature you don't understand
2. ✅ Perform impact analysis for a planned change
3. ✅ Create a data lineage map for your key tables

---

**Related Sections:**
- [Phase 1.4.12: Tool Use](./04-12-tool-use.md) - Tool-directed workflows
- [Phase 1.4.10: Complex Prompts](./04-10-complex-prompts.md) - Combining techniques
- [Phase 1.3.4: CLI Reference](../03-04-cli-reference.md) - Grep and Glob details

---

**Last Updated:** 2025-10-24
**Version:** 1.0
**Based on:** Anthropic Tutorial - Appendix 10.3: Search & Retrieval
