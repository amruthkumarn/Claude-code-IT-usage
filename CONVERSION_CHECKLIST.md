# PySpark/Python Conversion Checklist

**Status:** In Progress
**Started:** 2025-10-22
**Purpose:** Convert all JavaScript/TypeScript code examples to Python/PySpark for Banking IT Data Chapter

---

## ✅ Completed Files (2/17)

### 1. ✅ 02-basics/03-getting-started.md
**Changes Made:**
- Converted JavaScript payment processing example → Python PySpark transaction processing
- Updated `calculateInterest` → `calculate_interest_rate` UDF
- Changed `/api/accounts endpoint` → `customer account aggregation job`
- Converted async/await callbacks → PySpark DataFrame operations
- Updated all file paths: `src/auth/` → `pipelines/`, `utils/`, etc.
- Changed error types: `TypeError` → `AttributeError`
- Updated technology stack: Node.js/Express → PySpark/Databricks
- Changed testing framework: Jest → pytest
- Updated all project structure references

**Code Blocks Updated:** ~10
**File Path References Updated:** ~15

---

### 2. ✅ 03-advanced/06-memory-management.md
**Changes Made:**
- Converted "Payment Processing Service" → "Payment Data Processing Pipeline"
- TypeScript CLAUDE.md example → Python/PySpark CLAUDE.md
- Changed REST API authentication → Data access controls
- Updated error handling: try-catch → try-except with proper logging
- Changed testing: Mock API calls → Mock Spark sessions
- Database: Parameterized queries → Spark SQL with Delta Lake

**Code Blocks Updated:** 1 large example (~50 lines)

---

## 🔄 High Priority - Remaining (2 files)

### 3. ⏳ 03-advanced/07-slash-commands.md
**File Purpose:** Custom slash command examples
**Estimated Changes:** 20-30 file references, 5-10 code examples

**Changes Needed:**

#### Line 343: API Documentation Command
```markdown
# BEFORE
> /api-docs src/api/payments.ts

# AFTER
> /pipeline-docs pipelines/payment_processing.py
```

#### Lines 315-339: Generate API Documentation Command
```markdown
# CHANGE FROM:
Generate OpenAPI 3.0 documentation for: $1
- All endpoints (GET, POST, PUT, DELETE, PATCH)
- Request/response schemas
- Authentication requirements

# CHANGE TO:
Generate data pipeline documentation for: $1
- Pipeline inputs and outputs
- Data schemas (StructType definitions)
- Transformation steps
- Data quality checks
```

#### Lines 221-269: Compliance Check Command Example
- Change `src/auth/` → `pipelines/validators/`
- Change API security checks → Data pipeline security (PII handling, encryption)
- Update file paths throughout

#### Lines 346-391: Database Migration Generator
- Keep as-is (PostgreSQL migrations are still relevant)
- Update context to mention it's for metadata/config tables

#### Other Examples:
- `/test` command: Jest → pytest
- `/refactor` examples: JS files → .py files
- `/generate-tests` examples: API tests → PySpark transformation tests

---

### 4. ⏳ 03-advanced/16-prompt-engineering.md
**File Purpose:** Prompt engineering examples
**Estimated Changes:** 15-20 code examples

**Changes Needed:**

#### Find all JavaScript/TypeScript examples:
```bash
grep -n "```javascript\|```typescript\|```js\|```ts" 03-advanced/16-prompt-engineering.md
```

**Conversion Pattern:**
- Function syntax: `function name()` → `def name():`
- Const/let → Python variables
- API examples → Data pipeline examples
- Express routes → PySpark transformations
- Database queries → Spark SQL
- Error handling → Python try/except
- Type annotations → Python type hints

**Specific Sections:**
- **Chain of Thought examples:** Update code context
- **Few-shot learning examples:** Use Python/PySpark
- **Role prompting examples:** Data engineer instead of API developer
- **XML tags examples:** Keep structure, change code language

---

## 🔄 Medium Priority (5 files)

### 5. ⏳ 04-security/09-security-compliance.md
**Estimated Changes:** 5-10 code examples

**Changes Needed:**
- Lines ~134-147: Credential storage examples → Windows Credential Manager (already done)
- API security examples → Data pipeline security
- JWT validation → Data access validation
- SQL injection examples → Spark SQL injection prevention
- Error handling code → Python examples

**Specific Areas:**
- Authentication code → Data access control code
- API rate limiting → Pipeline throttling/resource management
- Session management → Spark session management

---

### 6. ⏳ 04-security/10-hooks-automation.md
**Estimated Changes:** 3-5 code examples, 5-10 file references

**Changes Needed:**
- Hook script examples using `.js/.ts` files → `.py` files
- npm scripts → Python/pytest scripts
- Example pre-commit hooks → Python-based hooks
- File paths: `src/` → `pipelines/`

**Example:**
```bash
# BEFORE
"command": "npm run lint"

# AFTER
"command": "ruff check pipelines/"
```

---

### 7. ⏳ 05-integration/11-mcp.md
**Estimated Changes:** 5-8 code examples

**Changes Needed:**
- API endpoint examples → Database/data source connections
- REST integration → Delta Lake, PostgreSQL, S3 integrations
- Server examples → Data source MCP servers
- File paths and technology references

**MCP Use Cases to Update:**
- Database MCP: API queries → Spark SQL queries
- Slack MCP: Keep as-is (notifications are universal)
- GitHub MCP: Update code references to Python
- Jira MCP: Keep as-is

---

### 8. ⏳ 05-integration/12-git-integration.md
**Estimated Changes:** 10-15 file path references

**Changes Needed:**
- Commit message examples with `.js/.ts` → `.py`
- Example: `src/auth/middleware.js` → `pipelines/validators/auth.py`
- Code review examples → Python code review
- PR description templates → Data engineering context

**Lines to Update:**
- ~250-300: Commit message examples
- ~350-400: PR creation examples
- File path references throughout

---

### 9. ⏳ 05-integration/13-standards-best-practices.md
**Estimated Changes:** Major rewrite (~50+ changes)

**Changes Needed:**

#### Technology Stack Section:
```markdown
# BEFORE
- TypeScript/JavaScript standards
- npm/package.json
- Jest testing
- ESLint/Prettier
- Airbnb style guide

# AFTER
- Python/PySpark standards
- pip/requirements.txt or poetry/pyproject.toml
- pytest testing
- Ruff/Black/mypy
- PEP 8 style guide
```

#### Code Quality Tools:
- ESLint → Ruff or Pylint
- Prettier → Black
- TypeScript → mypy
- Jest → pytest
- npm scripts → make, poetry scripts, or Python CLIs

#### Example Commands:
```bash
# BEFORE
npm run lint
npm test
npm run build

# AFTER
ruff check .
pytest tests/
spark-submit pipelines/main.py
```

---

## 🔄 Lower Priority (8 files)

### 10. ⏳ 06-reference/14-templates-library.md
**Estimated Changes:** 10-15 template examples

**Changes Needed:**
- API template → Data pipeline template
- Express routes → PySpark transformations
- Controller templates → Data processor templates
- Test templates: Jest → pytest

---

### 11. ⏳ 06-reference/15-troubleshooting-faq.md
**Estimated Changes:** 5-10 troubleshooting examples

**Changes Needed:**
- npm troubleshooting → pip/Python troubleshooting
- Node.js issues → Python/Spark issues
- Package installation → pip/poetry issues
- Module import errors → Python import errors

**Example:**
```markdown
# BEFORE
**Issue:** "npm ERR! peer dependency conflict"
**Solution:** npm install --legacy-peer-deps

# AFTER
**Issue:** "PIP dependency conflict"
**Solution:** pip install --upgrade --force-reinstall
```

---

### 12. ⏳ quick-reference/commands-cheatsheet.md
**Estimated Changes:** 3-5 command examples

**Changes Needed:**
- Example commands using npm → pip, pytest, spark-submit
- File path examples → Python paths
- Quick tips → Python/PySpark specific

**Commands to Update:**
```bash
# BEFORE
npm test
npm run lint

# AFTER
pytest tests/
ruff check .
```

---

### 13. ⏳ quick-reference/dos-and-donts.md
**Estimated Changes:** 10-15 references

**Changes Needed:**
- JavaScript/TypeScript dos and don'ts → Python/PySpark
- npm commands → pip/poetry commands
- API development tips → Data pipeline tips
- Technology-specific advice

**Examples:**
```markdown
# BEFORE
✅ DO: Use TypeScript for type safety
❌ DON'T: Use 'any' type

# AFTER
✅ DO: Use Python type hints and mypy
❌ DON'T: Skip type annotations on public functions
```

---

### 14. ⏳ templates/prompts/compliance-check-prompt.md
**Estimated Changes:** 2-3 file path references

**Changes Needed:**
- Line 60: `src/transactions/` → `pipelines/transaction_processing/`
- Context references to APIs → Data pipelines
- Code audit examples → PySpark audit examples

---

### 15. ⏳ templates/prompts/security-review-prompt.md
**Estimated Changes:** 5-10 code examples

**Changes Needed:**
- API security review → Data pipeline security review
- Authentication checks → Data access control checks
- SQL injection in APIs → Spark SQL injection
- Session management → Data access logging

**Example Prompts:**
```markdown
# BEFORE
Review API endpoints for:
- Authentication bypass
- Authorization issues
- SQL injection
- XSS vulnerabilities

# AFTER
Review data pipelines for:
- Data access control bypass
- PII exposure
- Spark SQL injection
- Data leakage
```

---

### 16. ⏳ 02-basics/04-cli-reference.md
**Estimated Changes:** 5-10 examples

**Changes Needed:**
- Lines ~566-594: Banking IT environment setup
  - Already updated for PowerShell ✓
- Example project paths → Python project paths
- npm commands → pip commands in examples

---

### 17. ⏳ 01-foundation/01-introduction-and-installation.md
**Estimated Changes:** Already mostly updated ✓

**Remaining:**
- Verify all examples use generic or Python context
- No JavaScript-specific references

---

## Conversion Reference Guide

### Quick Conversion Patterns

| JavaScript/TypeScript | Python/PySpark |
|----------------------|----------------|
| `function name() {}` | `def name():` |
| `const x = ...` | `x = ...` |
| `let x = ...` | `x = ...` (or just use reassignment) |
| `async/await` | `async/await` (if using asyncio) or DataFrame operations |
| `try { } catch (e) { }` | `try: ... except Exception as e: ...` |
| `throw new Error()` | `raise ValueError()` or `raise Exception()` |
| `.map()` / `.filter()` | `.select()` / `.filter()` (PySpark) or list comprehensions |
| `interface` / `type` | `@dataclass` / `TypedDict` / `StructType` |
| `src/api/` | `pipelines/` or `jobs/` |
| `src/auth/` | `utils/validators/` or `auth/` |
| `src/services/` | `services/` or `processors/` |
| `src/models/` | `schemas/` or `models/` |
| `.js` / `.ts` | `.py` |
| `.test.js` / `.spec.ts` | `_test.py` or `test_*.py` |
| `npm test` | `pytest` |
| `npm run lint` | `ruff check .` or `pylint` |
| `npm install` | `pip install` or `poetry install` |
| `package.json` | `requirements.txt` or `pyproject.toml` |
| `jest` | `pytest` |
| `ESLint` | `Ruff` or `Pylint` |
| `Prettier` | `Black` |
| `TypeScript` | `mypy` (type checking) |
| `Express` | `PySpark` / `FastAPI` (if APIs needed) |
| `Node.js` | `Python 3.9+` |
| REST API | Data Pipeline / Spark Job |
| endpoints | transformations |
| routes | data flows |
| middleware | data validators |
| controllers | processors |

### Banking IT Specific Mappings

| Generic | Banking Data Engineering |
|---------|-------------------------|
| User authentication | Data access validation |
| API rate limiting | Pipeline resource management |
| Session management | Spark session management |
| Request validation | Schema validation |
| Response formatting | DataFrame schema enforcement |
| JWT tokens | IAM roles / Kerberos |
| Payment processing API | Payment data transformation pipeline |
| Transaction endpoint | Transaction data processing job |
| Account management | Customer data pipeline |
| `calculateInterest()` | `calculate_interest_rate()` (UDF) |
| `validateUser()` | `validate_customer_data()` |
| `processPayment()` | `process_payment_transaction()` |

---

## Implementation Strategy

### Approach for Each File:

1. **Search for patterns:**
   ```bash
   grep -n "```javascript\|```typescript\|src/\|\.js\|\.ts" FILENAME.md
   ```

2. **Update code blocks:**
   - Change language tags: ` ```javascript` → ` ```python`
   - Convert syntax using reference guide above
   - Update function names to snake_case
   - Add type hints where appropriate

3. **Update file paths:**
   - Find: `src/api/`, `src/auth/`, etc.
   - Replace with: `pipelines/`, `utils/validators/`, etc.

4. **Update commands:**
   - npm → pip/poetry
   - jest → pytest
   - eslint → ruff

5. **Update technology references:**
   - Node.js → Python 3.9+
   - Express → PySpark
   - TypeScript → Python with type hints
   - PostgreSQL queries → Spark SQL

### Testing Strategy:

After each file update:
- [ ] Check all code blocks have correct language tags
- [ ] Verify file paths are consistent
- [ ] Ensure examples are relevant to data engineering
- [ ] Validate Python syntax (at least basic)
- [ ] Check for remaining JS/TS references

---

## Progress Tracking

**Total Files:** 17
**Completed:** 2 (12%)
**High Priority Remaining:** 2
**Medium Priority Remaining:** 5
**Lower Priority Remaining:** 8

### Estimated Time per File:
- High priority (complex): 30-45 minutes each
- Medium priority: 20-30 minutes each
- Lower priority (simple): 10-20 minutes each

**Total Estimated Time:** 5-7 hours

---

## Notes for Future Sessions

### Session Continuation Commands:

```bash
# Check progress
grep -r "```javascript\|```typescript" --include="*.md" . | wc -l

# Find remaining src/ references
grep -r "src/" --include="*.md" . | grep -v ".git"

# Find npm references
grep -r "npm " --include="*.md" . | grep -v ".git"
```

### Quick Validation:

```bash
# Count Python vs JavaScript code blocks
echo "Python blocks:" && grep -r "```python" --include="*.md" . | wc -l
echo "JavaScript blocks:" && grep -r "```javascript\|```typescript\|```js\|```ts" --include="*.md" . | wc -l
```

---

## Completion Criteria

The conversion is complete when:

- [ ] Zero JavaScript/TypeScript code blocks in documentation
- [ ] All `src/` paths converted to appropriate data engineering paths
- [ ] All npm commands converted to pip/pytest/poetry
- [ ] All API examples converted to data pipeline examples
- [ ] Technology stack references updated throughout
- [ ] All CLAUDE.md examples use Python/PySpark
- [ ] All slash command examples use Python files
- [ ] Validation scripts show no remaining JS/TS references

---

**Last Updated:** 2025-10-22
**Updated By:** Claude Code Session
**Next Session:** Continue with file #3 (07-slash-commands.md)
