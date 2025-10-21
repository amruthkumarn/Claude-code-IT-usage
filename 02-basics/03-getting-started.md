# Section 3: Getting Started

## Table of Contents
1. [Your First Claude Code Session](#your-first-claude-code-session)
2. [Understanding Your Codebase](#understanding-your-codebase)
3. [Fixing Bugs](#fixing-bugs)
4. [Refactoring Code](#refactoring-code)
5. [Using Plan Mode](#using-plan-mode)
6. [Keyboard Shortcuts](#keyboard-shortcuts)
7. [Help Commands](#help-commands)
8. [Common Workflows](#common-workflows)
9. [Practical Banking Examples](#practical-banking-examples)
10. [Best Practices for Beginners](#best-practices-for-beginners)

---

## Your First Claude Code Session

Let's walk through your very first Claude Code session step by step.

### Step 1: Navigate to Your Project

```bash
# Navigate to a project directory
cd ~/projects/my-banking-api

# Verify you're in the right place
pwd
ls
```

### Step 2: Start Claude Code

```bash
# Start the interactive REPL
claude
```

You'll see:
```
Welcome to Claude Code!
You're in: /Users/yourname/projects/my-banking-api

Type /help for available commands
Press ? for keyboard shortcuts

>
```

### Step 3: Ask Your First Question

```
> What is this project about?
```

Claude will:
1. Look for README files
2. Check package.json or similar configuration files
3. Scan the directory structure
4. Provide a summary

**Example Response:**
```
This appears to be a Node.js REST API for banking operations. Based on the structure:

- Technology: Node.js with Express
- Database: PostgreSQL (via pg library)
- Authentication: JWT tokens
- Main features:
  - Account management (src/accounts/)
  - Transaction processing (src/transactions/)
  - User authentication (src/auth/)
  - Payment processing (src/payments/)

The project has tests in the test/ directory and uses Jest as the testing framework.
```

### Step 4: Dive Deeper

```
> Can you show me how authentication is implemented?
```

Claude will read relevant files and explain the authentication flow.

### Step 5: Make a Change

```
> Can you add request rate limiting to the authentication endpoint?
```

Claude will:
1. Propose a solution
2. Show you the code changes
3. Ask for your approval
4. Apply the changes once approved

### Step 6: Exit

```bash
# Press Ctrl+D or type:
> /exit
```

---

## Understanding Your Codebase

One of Claude Code's most powerful features is helping you understand unfamiliar code.

### Getting an Overview

```
> Give me a high-level overview of this codebase
```

**What Claude does:**
- Reads directory structure
- Identifies key files and patterns
- Summarizes the architecture
- Notes dependencies and frameworks

### Finding Specific Code

```
> Where is the password hashing implemented?
```

Claude will search for relevant files and show you the implementation.

### Understanding a Specific File

```
> Explain what src/transactions/processor.js does
```

Claude will:
- Read the file
- Explain its purpose
- Describe key functions
- Note any potential issues

### Tracing Data Flow

```
> Show me how a payment request flows through the system, from the API endpoint to the database
```

Claude will:
- Find the entry point (API route)
- Trace through middleware
- Show business logic
- Follow to database operations

### Finding Dependencies

```
> What files depend on the UserService class?
```

Claude will search for imports and usages across the codebase.

### Banking Example: Understanding Legacy Code

```
> I need to understand the SWIFT payment processing logic.
> Can you explain how it works and identify any potential issues?
```

Claude will:
1. Find SWIFT-related code
2. Explain the processing flow
3. Identify validation logic
4. Flag potential issues (error handling, security, etc.)

---

## Fixing Bugs

Claude Code excels at debugging. Here's how to use it effectively.

### Scenario 1: Runtime Error

**You have an error:**
```
TypeError: Cannot read property 'accountId' of undefined
  at TransactionService.processPayment (src/transactions/service.js:45)
```

**Ask Claude:**
```
> I'm getting this error:
> TypeError: Cannot read property 'accountId' of undefined
> at TransactionService.processPayment (src/transactions/service.js:45)
>
> Can you help me fix it?
```

**Claude's approach:**
1. Reads the file at line 45
2. Examines the context
3. Identifies the root cause
4. Proposes a fix
5. Shows you the change

**Example fix:**
```javascript
// Before
processPayment(transaction) {
  const accountId = transaction.account.accountId;
  // ...
}

// After
processPayment(transaction) {
  if (!transaction || !transaction.account) {
    throw new Error('Invalid transaction: missing account information');
  }
  const accountId = transaction.account.accountId;
  // ...
}
```

### Scenario 2: Logic Error

```
> The calculateInterest function is returning incorrect values for accounts
> with balances over $10,000. Can you investigate?
```

Claude will:
1. Find the function
2. Analyze the logic
3. Test edge cases
4. Identify the bug
5. Propose a fix

### Scenario 3: Test Failures

```
> The test "should validate IBAN format" is failing. Can you fix it?
```

Claude will:
1. Read the test file
2. Read the implementation
3. Identify the mismatch
4. Fix either the test or implementation (or both)

### Scenario 4: Performance Issue

```
> The /api/accounts endpoint is very slow when there are many accounts.
> Can you optimize it?
```

Claude will:
1. Examine the endpoint code
2. Look for N+1 queries
3. Check for missing indexes
4. Suggest optimizations (pagination, caching, query optimization)

---

## Refactoring Code

Use Claude to improve code quality without changing behavior.

### Modernizing Code

```
> Refactor src/auth/legacy-validator.js to use modern JavaScript (async/await, ES6+)
```

**Before:**
```javascript
function validateUser(userId, callback) {
  db.query('SELECT * FROM users WHERE id = ?', [userId], function(err, results) {
    if (err) return callback(err);
    if (results.length === 0) return callback(new Error('User not found'));
    callback(null, results[0]);
  });
}
```

**After (Claude's suggestion):**
```javascript
async function validateUser(userId) {
  try {
    const results = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
    if (results.length === 0) {
      throw new Error('User not found');
    }
    return results[0];
  } catch (error) {
    throw new Error(`User validation failed: ${error.message}`);
  }
}
```

### Extracting Duplicate Code

```
> I see a lot of duplicate validation logic in the account controllers.
> Can you extract it into reusable functions?
```

Claude will:
1. Identify the duplicate patterns
2. Create a shared validation module
3. Update all files to use the new module

### Improving Error Handling

```
> Add comprehensive error handling to src/payments/processor.js
```

Claude will add:
- Try-catch blocks
- Proper error messages
- Error logging
- Recovery strategies

### Adding Type Safety

```
> Add TypeScript types to src/transactions/types.js
```

Claude will:
- Create proper TypeScript interfaces
- Add type annotations
- Fix type errors

---

## Using Plan Mode

Plan Mode is a **read-only** mode perfect for exploring code safely.

### What is Plan Mode?

- No write operations allowed
- No command execution
- Perfect for understanding unfamiliar code
- Safe to use on production codebases

### Starting Plan Mode

```bash
# Start Claude in plan mode
claude --permission-mode plan
```

You'll see:
```
Claude Code - Plan Mode (Read-only)
No changes will be made to your files

>
```

### When to Use Plan Mode

**Use Plan Mode when:**
- Exploring a new codebase
- Understanding legacy systems
- Code reviews (read-only)
- Learning how something works
- You want to ask questions without risk

**Exit Plan Mode for:**
- Making actual changes
- Running tests
- Creating commits

### Plan Mode Example

```bash
# Start in plan mode
claude --permission-mode plan

> Analyze the security of the authentication system
>
> Claude will read and analyze but cannot make changes

> Create a report of potential security issues
```

Then exit and start a normal session to make fixes:

```bash
# Exit plan mode
Ctrl+D

# Start normal mode to make changes
claude

> Based on the security analysis, let's fix issue #1: SQL injection vulnerability
```

### Banking Use Case: Auditing

```bash
# Audit code before deployment
cd ~/projects/payment-processor
claude --permission-mode plan

> Review all database queries for SQL injection vulnerabilities
> Check for proper error handling in payment processing
> Verify authentication is required on all endpoints
> Look for any hardcoded credentials or secrets
```

---

## Keyboard Shortcuts

Master these shortcuts for efficient usage.

### Essential Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+D` | Exit Claude Code |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+L` | Clear screen |
| `↑` / `↓` | Navigate command history |
| `?` | Show help and shortcuts |

### Input Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Autocomplete (when available) |
| `Ctrl+A` | Move to start of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+K` | Delete from cursor to end of line |
| `Ctrl+U` | Delete from cursor to start of line |

### During Approval Prompts

| Key | Action |
|-----|--------|
| `A` | Approve action |
| `R` | Reject action |
| `E` | Edit before applying |
| `V` | View full context |
| `?` | Show more options |

---

## Help Commands

Get help anytime during your session.

### Built-in Help

```bash
> /help
```

Shows all available commands:
```
Available Commands:
  /help          Show this help message
  /clear         Clear conversation history
  /model         Change AI model (sonnet, opus, haiku)
  /config        Open settings
  /login         Login or check authentication status
  /memory        Edit memory files (CLAUDE.md)
  /review        Request code review
  /exit          Exit Claude Code

Press ? for keyboard shortcuts
```

### Getting Specific Help

```bash
> /help model
# Shows detailed help about the /model command

> /help config
# Shows help about configuration
```

### Quick Tips

```bash
> ?
```

Shows quick tips and keyboard shortcuts.

### Documentation

For comprehensive documentation, visit:
- **Official Docs**: https://docs.claude.com/en/docs/claude-code/overview
- **Quickstart Guide**: https://docs.claude.com/en/docs/claude-code/quickstart
- **Common Workflows**: https://docs.claude.com/en/docs/claude-code/common-workflows

---

## Common Workflows

### Workflow 1: Adding a New Feature

```
Step 1: Understand existing code
> Show me how the account creation feature works currently

Step 2: Plan the new feature
> I need to add a feature for joint accounts (multiple owners).
> What changes would be needed?

Step 3: Implement
> Let's implement the joint accounts feature.
> Start with the database schema changes.

Step 4: Add tests
> Now add tests for the joint account functionality

Step 5: Review
> /review
> Review the changes we just made for potential issues
```

### Workflow 2: Bug Investigation and Fix

```
Step 1: Reproduce
> I'm seeing incorrect interest calculations for accounts
> with compound interest. Can you help reproduce this?

Step 2: Investigate
> Find all code related to interest calculation

Step 3: Identify root cause
> The bug appears when calculating daily compound interest
> for partial months. Analyze the calculateCompoundInterest function

Step 4: Fix
> Fix the bug and add a test case to prevent regression

Step 5: Verify
> Run the tests to verify the fix works
```

### Workflow 3: Code Review

```bash
# Use plan mode for read-only review
claude --permission-mode plan

> Review the changes in src/payments/ for:
> - Security issues
> - Error handling
> - Code quality
> - Potential bugs
> - Performance concerns
>
> Provide a detailed report
```

### Workflow 4: Documentation

```
> Generate documentation for the PaymentProcessor class
> Include:
> - Class overview
> - Method descriptions
> - Parameter types
> - Return values
> - Example usage
> - Error conditions
```

### Workflow 5: Onboarding to New Project

```
Day 1 - Overview:
> What is this project? What's its purpose?
> What are the main technologies used?
> Show me the project structure

Day 1 - Setup:
> What do I need to install to run this locally?
> Show me the development workflow

Day 2 - Deep Dive:
> Explain the authentication system
> How does the payment processing work?
> Where are the API endpoints defined?

Day 3 - Contributing:
> What are the coding standards for this project?
> Show me how to run the tests
> What's the git workflow?
```

---

## Practical Banking Examples

### Example 1: IBAN Validation

```
> I need to add IBAN validation to the account creation endpoint.
> The validation should:
> - Check IBAN format
> - Validate country code
> - Verify check digits
> - Support all EU country formats
>
> Create a reusable validator function with tests
```

Claude will:
1. Create a validation function
2. Add country-specific rules
3. Implement check digit validation
4. Write comprehensive tests
5. Integrate with the endpoint

### Example 2: Transaction Audit Logging

```
> Add audit logging for all financial transactions.
>
> Requirements:
> - Log user ID, timestamp, action, amount
> - Include before/after balances
> - Store in separate audit table
> - Never fail the transaction if logging fails
> - Comply with SOX requirements
```

Claude will:
1. Create audit log schema
2. Implement logging service
3. Add to transaction flow
4. Handle logging failures gracefully
5. Add compliance documentation

### Example 3: Rate Limiting

```
> Add rate limiting to prevent brute force attacks on the login endpoint.
>
> Requirements:
> - Max 5 attempts per IP per 15 minutes
> - Return 429 status when exceeded
> - Log suspicious activity
> - Whitelist internal IPs
```

Claude will:
1. Implement rate limiting middleware
2. Use Redis for distributed rate limiting
3. Add IP whitelist logic
4. Implement logging
5. Add tests

### Example 4: Currency Conversion

```
> Create a currency conversion service for international transfers.
>
> Requirements:
> - Support 20+ currencies
> - Get real-time exchange rates from external API
> - Cache rates for 5 minutes
> - Include margin calculation
> - Handle API failures gracefully
```

Claude will:
1. Create service structure
2. Implement API integration
3. Add caching layer
4. Handle errors and fallbacks
5. Write tests with mocked API

### Example 5: Legacy System Integration

```
> We need to integrate with a legacy COBOL mainframe system.
> The integration uses IBM MQ for messaging.
>
> Can you:
> 1. Show me existing MQ integrations in the codebase
> 2. Create a service to send account updates to the mainframe
> 3. Handle message formatting (fixed-width fields)
> 4. Add error handling and retry logic
```

Claude will:
1. Find existing patterns
2. Create integration service
3. Implement message formatting
4. Add robust error handling
5. Document the integration

---

## Best Practices for Beginners

### 1. Start Small

Don't try to refactor your entire codebase on day one.

**Good first tasks:**
- Ask questions about specific files
- Fix a single bug
- Add comments to undocumented code
- Write a test for existing functionality

### 2. Be Specific

**Vague:**
```
> Fix the bugs
```

**Specific:**
```
> The login endpoint returns 500 error when the password field is missing.
> Can you add validation and return 400 with a clear error message?
```

### 3. Review Everything

Always review Claude's suggestions before approving:
- Do the changes make sense?
- Are there any security implications?
- Will this affect other parts of the system?
- Are tests needed?

### 4. Use Plan Mode for Learning

When learning a new codebase, start in Plan Mode:

```bash
claude --permission-mode plan

> Explain how the authentication system works
> Show me the database schema
> Trace a payment transaction through the system
```

### 5. Break Down Complex Tasks

**Instead of:**
```
> Rewrite the entire payment system to use microservices
```

**Do:**
```
> Step 1: Show me the current payment system architecture
> Step 2: Identify components that could be extracted
> Step 3: Create a plan for migrating to microservices
> Step 4: Let's start by extracting the payment validation logic
```

### 6. Keep Context Focused

If Claude seems to "forget" earlier information:

```bash
# Start a fresh session
Ctrl+D
claude

> Let's focus on just the authentication module
```

### 7. Use Memory for Project Standards

Create a CLAUDE.md file in your project:

```markdown
# Project Standards

## Code Style
- Use TypeScript
- Follow Airbnb style guide
- Always use async/await

## Security
- Never log sensitive data
- Always use parameterized queries
- Require authentication on all endpoints

## Testing
- Minimum 80% coverage
- Test error cases
- Mock external dependencies
```

Now Claude will follow these standards automatically.

### 8. Leverage the /review Command

Before committing:

```
> /review

> Review the changes we made for:
> - Security issues
> - Code quality
> - Edge cases
> - Test coverage
```

### 9. Document As You Go

```
> Add JSDoc comments to all public functions in this file
```

This helps both humans and Claude understand the code better.

### 10. Use Version Control

Always work in a git branch:

```bash
git checkout -b feature/add-rate-limiting
claude

> Add rate limiting to the login endpoint

# Review changes
git diff

# Commit if satisfied
git add .
git commit -m "Add rate limiting to login endpoint"
```

---

## Common Mistakes to Avoid

### Mistake 1: Starting in Root Directory

**Don't:**
```bash
cd ~
claude  # Can access everything in home directory!
```

**Do:**
```bash
cd ~/projects/specific-project
claude  # Limited to project scope
```

### Mistake 2: Auto-Approving Everything

**Don't:**
```bash
claude --permission-mode auto-approve
# Dangerous! Claude can make any changes without review
```

**Do:**
```bash
claude
# Review each change individually
```

### Mistake 3: Vague Requests

**Don't:**
```
> Make it better
```

**Do:**
```
> Refactor the validateUser function to:
> - Use async/await instead of callbacks
> - Add input validation
> - Improve error messages
> - Add JSDoc comments
```

### Mistake 4: Ignoring Context Limits

If Claude starts "forgetting" things:

**Don't:**
```
> Remember earlier when we talked about... [repeat entire history]
```

**Do:**
```
# Start fresh session
Ctrl+D
claude

> Let's focus on optimizing the payment processor
```

### Mistake 5: Not Using Plan Mode for Exploration

**Don't:**
```bash
# In unfamiliar codebase
claude

> Change all API endpoints to use GraphQL
# Risk of breaking changes!
```

**Do:**
```bash
claude --permission-mode plan

> Analyze the current API structure
> What would be involved in migrating to GraphQL?

# Exit and review findings before making changes
```

---

## Next Steps

Now that you know the basics:

1. **Practice**: Try the workflows with your actual codebase
2. **[Continue to Section 4: CLI Reference](./04-cli-reference.md)** - Learn all command-line options
3. **[Jump to Section 5: Project Configuration](../03-advanced/05-project-configuration.md)** - Configure for your team
4. **[Review Common Workflows](https://docs.claude.com/en/docs/claude-code/common-workflows)** - More workflow examples

---

## Summary

In this section, you learned:

### Core Skills
- Starting and using Claude Code sessions
- Understanding unfamiliar codebases
- Fixing bugs with AI assistance
- Refactoring code safely
- Using Plan Mode for exploration

### Practical Knowledge
- Keyboard shortcuts for efficiency
- Help commands and documentation
- Common development workflows
- Banking-specific examples
- Best practices for beginners

### Key Takeaways
1. Start with small, specific tasks
2. Always review changes before approving
3. Use Plan Mode for safe exploration
4. Break complex tasks into steps
5. Create memory files for project standards
6. Keep Claude's context focused
7. Work in git branches for safety

---


**Additional Resources:**
- **Claude Code Documentation**: https://docs.claude.com/en/docs/claude-code/overview
- **Common Workflows Guide**: https://docs.claude.com/en/docs/claude-code/common-workflows
- **Interactive Mode Reference**: https://docs.claude.com/en/docs/claude-code/interactive-mode
