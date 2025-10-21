# Claude Code - Dos and Don'ts for Banking IT Developers

**Target Audience:** Banking IT - Data Chapter
**Version:** 1.0
**Last Updated:** 2025-10-21

---

## Introduction

This guide provides critical dos and don'ts for using Claude Code in banking IT environments. Following these guidelines ensures:

- **Security**: Protect sensitive banking data and systems
- **Compliance**: Meet SOX, PCI-DSS, and GDPR requirements
- **Quality**: Maintain high code quality and standards
- **Efficiency**: Use Claude Code effectively
- **Accountability**: Maintain proper audit trails

🔒 **Banking IT Context**: All guidelines prioritize security, compliance, and audit requirements specific to financial services.

---

## Table of Contents

1. [Security & Compliance](#security--compliance)
2. [Git Operations (Banking Policy)](#git-operations-banking-policy)
3. [Project Scope & Access Control](#project-scope--access-control)
4. [Prompting & Communication](#prompting--communication)
5. [Configuration & Setup](#configuration--setup)
6. [Data Protection & Privacy](#data-protection--privacy)
7. [Performance & Context Management](#performance--context-management)
8. [Code Review & Quality](#code-review--quality)
9. [Team Collaboration](#team-collaboration)
10. [Model Usage](#model-usage)

---

## Security & Compliance

### ✅ DO

**1. Always use plan mode for unfamiliar codebases**
```bash
claude --permission-mode plan
> Analyze the payment processing system
```
**Why**: Read-only exploration prevents accidental changes to critical systems.

**2. Review EVERY change before approving**
```bash
# When Claude proposes changes
[R] Reject if unsure
[V] View full context
[A] Approve only after thorough review
```
**Why**: Human oversight is required for compliance and prevents errors.

**3. Deny Bash tool in production-adjacent environments**
```json
{
  "permissions": {
    "deny": ["Bash"]
  }
}
```
**Why**: Prevents command execution, including git automation (not approved for banking).

**4. Enable audit logging for all Claude Code sessions**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": ".*",
      "hooks": [{
        "type": "command",
        "command": "./scripts/audit-log.sh"
      }]
    }]
  }
}
```
**Why**: SOX and PCI-DSS require audit trails of all code modifications.

**5. Use hooks to detect secrets before approval**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "./.claude/hooks/detect-secrets.sh"
      }]
    }]
  }
}
```
**Why**: Prevents accidental commits of API keys, passwords, tokens.

**6. Configure strict permissions in .claude/settings.json**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash", "WebFetch"]
  }
}
```
**Why**: Least privilege principle - only allow what's necessary.

**7. Document security requirements in CLAUDE.md**
```markdown
## Security Requirements
- Never commit secrets
- All database queries must be parameterized
- Authentication required for all endpoints
- Audit logging for financial transactions
```
**Why**: Claude follows project-specific security standards.

**8. Review for PII/sensitive data before every approval**
**Why**: GDPR and data protection regulations require explicit handling of personal data.

**9. Use separate environments for Claude Code**
```bash
# Development only
cd ~/dev/project
claude

# NEVER in production
```
**Why**: Prevents production impact, maintains separation of environments.

**10. Regularly review Claude Code settings and hooks**
```bash
# Monthly review
cat .claude/settings.json
ls -la .claude/hooks/
```
**Why**: Ensures configurations remain secure and compliant.

### ❌ DON'T

**1. NEVER use auto-approve mode**
```bash
# FORBIDDEN
claude --permission-mode auto-approve
```
**Why**: Bypasses all approval workflows, violates compliance requirements.

**2. NEVER skip security review of changes**
```bash
# BAD: Approving without reading
[A] [A] [A]  # Blind approval
```
**Why**: May introduce vulnerabilities or violate security policies.

**3. NEVER commit secrets, API keys, or credentials**
```javascript
// DON'T
const API_KEY = "sk-ant-abc123...";  // Hardcoded secret

// DO
const API_KEY = process.env.ANTHROPIC_API_KEY;  // From environment
```
**Why**: PCI-DSS violation, security breach risk.

**4. NEVER disable security hooks**
```json
// DON'T
{
  "hooks": {}  // Disabled
}
```
**Why**: Removes critical safety checks.

**5. NEVER use Claude Code with elevated privileges**
```bash
# FORBIDDEN
sudo claude
```
**Why**: Security risk, can modify system files.

**6. NEVER ignore security warnings from hooks**
```bash
# Hook warns: "Potential secret detected"
# DON'T: Approve anyway
# DO: Investigate and fix
```
**Why**: Warnings indicate real security issues.

**7. NEVER share Claude Code sessions with sensitive data**
**Why**: Sessions may contain PII, credentials, or proprietary code.

**8. NEVER use Claude Code on shared/public computers**
**Why**: Session history and credentials may be exposed.

---

## Git Operations (Banking Policy)

### ✅ DO

**1. Execute ALL git commands manually**
```bash
# Proper workflow
claude
> Fix the bug

Ctrl+D
git diff          # Manual review
git add .         # Manual staging
git commit -m ""  # Manual commit
git push          # Manual push
```
**Why**: Banking IT policy - git automation NOT approved.

**2. Ask Claude for commit messages (then use manually)**
```bash
claude --permission-mode plan
> Draft commit message for login bug fix

# Claude suggests message
# YOU copy and execute:
git commit -m "fix(auth): prevent session timeout on active users"
```
**Why**: Leverages Claude's expertise while maintaining manual control.

**3. Review git diff before every commit**
```bash
git diff          # See all changes
git diff --staged # See staged changes
```
**Why**: Catch unintended changes, secrets, debugging code.

**4. Use Claude to generate PR descriptions (paste manually)**
```bash
claude --permission-mode plan
> Generate PR description for transaction validation feature

# Copy output to GitHub PR manually
```
**Why**: High-quality PR descriptions while maintaining manual workflow.

**5. Document git policy in CLAUDE.md**
```markdown
## Git Operations Policy
CRITICAL: Manual Git Only
- Claude NEVER executes git commands
- All git operations performed manually
- Claude can suggest messages/descriptions
```
**Why**: Ensures all team members follow the same policy.

**6. Verify Claude cannot execute git commands**
```json
{
  "permissions": {
    "deny": ["Bash"]
  }
}
```
**Why**: Enforces policy at configuration level.

**7. Always exit Claude before git operations**
```bash
claude
> Make changes
Ctrl+D  # Exit Claude
git add .
git commit
```
**Why**: Clear separation between code changes and version control.

**8. Use conventional commit format (with Claude's help)**
```bash
claude --permission-mode plan
> Draft conventional commit message for payment integration
```
**Why**: Consistent commit history, easier to track changes.

### ❌ DON'T

**1. NEVER let Claude execute git commands**
```bash
# FORBIDDEN
claude
> Create a commit for these changes  # NO!
> Push to remote                      # NO!
```
**Why**: Violates banking IT policy, compromises audit trail.

**2. NEVER use git automation tools with Claude**
```bash
# FORBIDDEN
claude --agents '{
  "git-bot": {
    "tools": ["Bash"]  # Can run git commands
  }
}'
```
**Why**: Bypasses manual git policy.

**3. NEVER skip reviewing git diff**
```bash
# BAD
git add .
git commit -m "updates"  # Without reviewing
```
**Why**: May commit secrets, bugs, or unintended changes.

**4. NEVER commit directly to main/master**
```bash
# BAD
git checkout main
git commit
git push
```
**Why**: Violates branch protection, skips code review.

**5. NEVER use vague commit messages**
```bash
# BAD
git commit -m "fixed stuff"
git commit -m "updates"

# GOOD (with Claude's help)
git commit -m "fix(auth): prevent session timeout on active users

- Add heartbeat mechanism to keep sessions alive
- Update session TTL to 30 minutes
- Add logging for timeout events

Compliance: SEC-4.2"
```
**Why**: Poor audit trail, hard to track changes.

**6. NEVER commit without running tests**
```bash
# Before commit
npm test           # Or your test command
git add .
git commit
```
**Why**: May introduce bugs to codebase.

**7. NEVER force push to shared branches**
```bash
# FORBIDDEN
git push --force origin main
```
**Why**: Can destroy team's work, violates audit trail.

---

## Project Scope & Access Control

### ✅ DO

**1. Always start Claude in specific project directory**
```bash
# GOOD
cd ~/projects/payment-service
claude

# Shows: Working directory: /Users/you/projects/payment-service
```
**Why**: Limits Claude's access to relevant code only.

**2. Use --add-dir for related directories only**
```bash
cd ~/projects/frontend
claude --add-dir ../backend-api --add-dir ../shared-types
```
**Why**: Access to necessary dependencies without full filesystem access.

**3. Verify working directory before starting**
```bash
pwd  # Check you're in right place
ls   # Verify directory contents
claude
```
**Why**: Prevents accidental access to wrong codebase.

**4. Use separate Claude sessions for separate projects**
```bash
# Terminal 1: Payment service
cd ~/projects/payment-service && claude

# Terminal 2: Auth service (different session)
cd ~/projects/auth-service && claude
```
**Why**: Prevents context mixing, maintains isolation.

**5. Document scope in CLAUDE.md**
```markdown
## Project Scope
This project includes:
- src/: Application code
- tests/: Test files
- docs/: Documentation

Does NOT include:
- Database credentials (use env vars)
- Production configs
```
**Why**: Clear boundaries for Claude's awareness.

### ❌ DON'T

**1. NEVER start Claude in home directory**
```bash
# FORBIDDEN
cd ~
claude  # Can access all your files!
```
**Why**: Gives Claude access to entire filesystem, personal files.

**2. NEVER use Claude in directories with sensitive data**
```bash
# DON'T
cd ~/Documents/customer-data
claude
```
**Why**: Risk of PII exposure, compliance violation.

**3. NEVER give Claude access to production credentials**
```bash
# DON'T
cd ~/production-configs
claude
```
**Why**: Security breach risk.

**4. NEVER use --add-dir for unrelated projects**
```bash
# BAD
claude --add-dir ~/projects/project-a \
       --add-dir ~/projects/project-b \
       --add-dir ~/projects/project-c
```
**Why**: Unnecessary context, potential for cross-contamination.

**5. NEVER navigate outside project during session**
```bash
# In Claude session - DON'T
> Can you check my file at ~/Documents/personal.txt
```
**Why**: Claude should stay within project scope.

---

## Prompting & Communication

### ✅ DO

**1. Be specific and detailed in requests**
```
❌ Fix the bug

✅ The login endpoint returns 500 when password is missing.
   Add validation to check for password field.
   Return 400 with error: "Password is required"
   Add unit test for this case.
```
**Why**: Specific requests get better results.

**2. Provide context about your codebase**
```
✅ Add JWT authentication to API endpoints.
   We use Express.js with TypeScript.
   Existing auth is in src/auth/middleware.ts
   Token expiry: 15 minutes
   Follow the pattern in that file.
```
**Why**: Claude adapts to your specific setup.

**3. Request structured output formats**
```
✅ Scan for SQL injection vulnerabilities and return JSON:
   {
     "findings": [
       {"file": "path", "line": 42, "severity": "high", "description": "..."}
     ]
   }
```
**Why**: Easier to process, integrate with tools.

**4. Use chain of thought for complex problems**
```
✅ Debug the transaction rollback issue.
   Think step by step:
   1. Identify where transactions start
   2. Trace error handling
   3. Check rollback calls
   4. Verify database connection handling
   5. Suggest fix with explanation
```
**Why**: Better reasoning, more thorough analysis.

**5. Reference specific files and line numbers**
```
✅ In src/payments/processor.ts lines 45-67, the error handling
   doesn't catch network timeouts. Please add timeout handling.
```
**Why**: Precise location, faster resolution.

**6. Ask for explanations before accepting changes**
```
✅ Before you make changes, explain:
   1. What you're going to change
   2. Why this fixes the issue
   3. Any potential side effects
   4. What tests should be added
```
**Why**: Understand the changes, learn in the process.

**7. Request security and compliance checks**
```
✅ Implement password reset, ensuring:
   - Tokens expire after 1 hour
   - Rate limiting (3 attempts/hour)
   - Audit logging of all attempts
   - PCI-DSS compliant token generation
```
**Why**: Bakes security into requirements.

**8. Use examples to clarify intent**
```
✅ Add input validation like this:
   ```typescript
   if (!email || !email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
     throw new ValidationError("Invalid email");
   }
   ```
   Apply similar validation to all user inputs.
```
**Why**: Shows exact expected outcome.

### ❌ DON'T

**1. NEVER use vague requests**
```
❌ Make it better
❌ Fix everything
❌ Optimize the code
```
**Why**: Unclear intent, unpredictable results.

**2. NEVER ask Claude to handle what you should review**
```
❌ Figure out what's wrong and fix it
```
**Why**: You should understand the problem first.

**3. NEVER paste production data or PII**
```
❌ Here's a customer record with issue:
   {id: 123, name: "John Doe", ssn: "123-45-6789", ...}
```
**Why**: GDPR violation, privacy breach.

**4. NEVER ask for generated secrets in production**
```
❌ Generate a production API key
❌ Create a JWT secret for production
```
**Why**: Secrets should be generated securely, not by AI.

**5. NEVER ask Claude to make architectural decisions alone**
```
❌ Decide whether we should use microservices or monolith

✅ Here are our requirements [list]. What are pros/cons of
   microservices vs monolith for our use case? I'll make final decision.
```
**Why**: Critical decisions need human judgment.

**6. NEVER ignore Claude's warnings**
```
Claude: "⚠️ This change might break existing functionality"
❌ Approve anyway
```
**Why**: Warnings indicate real risks.

**7. NEVER ask Claude to circumvent security**
```
❌ Disable authentication for testing
❌ Skip input validation
❌ Allow SQL queries without parameterization
```
**Why**: Creates security vulnerabilities.

---

## Configuration & Setup

### ✅ DO

**1. Create .claude directory for every project**
```bash
mkdir -p .claude/{commands,hooks,output-styles}
```
**Why**: Consistent project structure, team configuration.

**2. Configure permissions in settings.json**
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"],
    "deny": ["Bash"]
  }
}
```
**Why**: Enforces security policy at project level.

**3. Document standards in CLAUDE.md**
```markdown
# Payment Service

## Coding Standards
- TypeScript strict mode
- ESLint configuration in .eslintrc
- Prettier for formatting
- 80% test coverage minimum

## Security Requirements
- All inputs validated
- Parameterized queries only
- JWT tokens expire in 15 minutes
```
**Why**: Claude follows your team's standards.

**4. Add .claude/settings.local.json to .gitignore**
```gitignore
# Claude Code
.claude/settings.local.json
.claude/.sessions/
```
**Why**: Personal settings shouldn't be committed.

**5. Version control team configurations**
```bash
git add .claude/settings.json
git add .claude/CLAUDE.md
git add .claude/commands/
git add .claude/hooks/
git commit -m "chore: Add Claude Code team configuration"
```
**Why**: Consistent setup across team.

**6. Document custom commands in README**
```markdown
## Claude Code Commands
- `/compliance-check` - PCI-DSS/SOX compliance audit
- `/security-review` - Security vulnerability scan
- `/generate-tests` - Generate unit tests
```
**Why**: Team knows what commands are available.

**7. Set default model in settings**
```json
{
  "defaultModel": "sonnet"
}
```
**Why**: Consistent model usage across team.

**8. Create project-specific output styles**
```bash
.claude/output-styles/banking.md
```
**Why**: Consistent output formatting for reports.

### ❌ DON'T

**1. NEVER commit .claude/settings.local.json**
**Why**: Contains personal preferences, may contain sensitive paths.

**2. NEVER skip configuration setup**
```bash
# BAD
cd new-project
claude  # No .claude directory, no standards
```
**Why**: Missing standards, inconsistent behavior.

**3. NEVER use default settings without review**
**Why**: May not meet banking security requirements.

**4. NEVER hardcode paths in settings.json**
```json
// DON'T
{
  "env": {
    "DB_PATH": "/Users/john/databases/prod.db"
  }
}
```
**Why**: Won't work for other team members.

**5. NEVER disable all permissions**
```json
// DON'T - Claude can't do anything
{
  "permissions": {
    "deny": ["*"]
  }
}
```
**Why**: Claude becomes unusable.

**6. NEVER mix personal and team settings in settings.json**
**Why**: Use settings.local.json for personal preferences.

---

## Data Protection & Privacy

### ✅ DO

**1. Review for PII before every approval**
```bash
# Before approving changes, check for:
- Names, addresses, emails
- Phone numbers, SSNs
- Credit card numbers
- Account numbers
```
**Why**: GDPR requires explicit PII handling.

**2. Use environment variables for sensitive config**
```javascript
// DO
const DB_PASSWORD = process.env.DB_PASSWORD;

// DON'T
const DB_PASSWORD = "MySecretPassword123";
```
**Why**: Prevents credential exposure.

**3. Mask sensitive data in examples**
```
✅ When asking Claude for help:
"User with email us***@example.com is getting error 500"

Not:
"User user@real-email.com is getting error 500"
```
**Why**: Protects real user privacy.

**4. Use sample/synthetic data for testing**
```javascript
// DO - Sample data
const testUser = {
  email: "test@example.com",
  name: "Test User",
  id: "test-123"
};
```
**Why**: No real PII exposed.

**5. Enable secret detection hooks**
```bash
.claude/hooks/detect-secrets.sh
```
**Why**: Automated detection of credentials.

**6. Audit log all data access**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Read.*customer|Read.*payment",
      "hooks": [{
        "type": "command",
        "command": "./scripts/log-data-access.sh"
      }]
    }]
  }
}
```
**Why**: Compliance requirement for data access.

**7. Classify data in CLAUDE.md**
```markdown
## Data Classification
- Public: Documentation, public APIs
- Internal: Business logic, internal tools
- Confidential: Customer data, financial transactions
- Restricted: Authentication credentials, encryption keys

Claude Code should ONLY process Public and Internal data.
```
**Why**: Clear data handling boundaries.

**8. Use read-only mode for production data analysis**
```bash
# If you MUST analyze production logs (redacted)
claude --permission-mode plan
```
**Why**: Prevents accidental modifications.

### ❌ DON'T

**1. NEVER paste real customer data into Claude**
```
❌ > Analyze this transaction:
   {customerId: "REAL-ID", amount: 1000.00, ssn: "123-45-6789"}
```
**Why**: Privacy violation, compliance breach.

**2. NEVER use production credentials in development**
```javascript
// DON'T
const PROD_API_KEY = "sk-prod-abc123...";
```
**Why**: Security breach risk.

**3. NEVER commit .env files**
```bash
# DON'T
git add .env
```
**Why**: Contains secrets and credentials.

**4. NEVER log sensitive data**
```javascript
// DON'T
console.log("User password:", password);
console.log("Credit card:", cardNumber);
```
**Why**: Logs may be accessed, stored, or transmitted.

**5. NEVER process PII without explicit consent**
**Why**: GDPR requirement.

**6. NEVER share Claude Code sessions with PII**
**Why**: May leak data to unauthorized parties.

**7. NEVER use real email addresses in test data**
```javascript
// DON'T
const testEmail = "john.doe@realcompany.com";

// DO
const testEmail = "test@example.com";
```
**Why**: May send emails to real people.

**8. NEVER bypass data classification policies**
**Why**: Compliance violation.

---

## Performance & Context Management

### ✅ DO

**1. Start fresh sessions regularly**
```bash
# Every 1-2 hours or when changing tasks
Ctrl+D
claude
```
**Why**: Prevents context overflow, improves performance.

**2. Keep sessions focused on one task**
```
✅ Session 1: Fix authentication bug
✅ Session 2: Add payment integration
✅ Session 3: Write tests

Not:
❌ One session: Fix bug, add feature, write docs, refactor, ...
```
**Why**: Better performance, clearer context.

**3. Be specific to reduce context needs**
```
✅ Fix the validation in src/auth/validator.ts line 45

Not:
❌ Look through all files and find validation issues
```
**Why**: Less context = faster responses.

**4. Use plan mode for exploration**
```bash
claude --permission-mode plan
> Analyze the codebase structure
> Explain the payment flow
```
**Why**: Read-only, efficient for understanding.

**5. Exit and restart if Claude seems slow**
```bash
Ctrl+D
claude
```
**Why**: Clears accumulated context.

**6. Limit --add-dir usage**
```bash
# GOOD: Only what's needed
claude --add-dir ../shared-types

# BAD: Too much
claude --add-dir ../proj1 --add-dir ../proj2 --add-dir ../proj3
```
**Why**: More directories = more context = slower.

### ❌ DON'T

**1. NEVER keep sessions running for hours**
**Why**: Accumulates context, degrades performance.

**2. NEVER ask Claude to analyze entire codebase at once**
```
❌ Explain everything in this project
```
**Why**: Context overflow, slow performance.

**3. NEVER repeat information unnecessarily**
```
❌ Remember earlier I told you about X, then Y, then Z...
   [repeating entire conversation]
```
**Why**: Wastes context, start fresh instead.

**4. NEVER ignore performance degradation**
```
If responses slow down:
- Start new session
- Be more specific
- Reduce scope
```
**Why**: Performance issues compound over time.

---

## Code Review & Quality

### ✅ DO

**1. Review every change for security**
```bash
# When Claude proposes changes, check:
- Input validation present?
- SQL parameterized?
- Error handling secure?
- No secrets exposed?
```
**Why**: Catch vulnerabilities before commit.

**2. Ask for tests with every feature**
```
✅ Implement password reset functionality.
   Include unit tests for:
   - Valid reset flow
   - Expired tokens
   - Invalid tokens
   - Rate limiting
```
**Why**: Maintains test coverage, catches bugs.

**3. Request documentation updates**
```
✅ Add the new /api/reset-password endpoint.
   Update API documentation in docs/api.md
   Add JSDoc comments to functions
```
**Why**: Keeps documentation in sync.

**4. Verify compliance requirements met**
```
✅ Before approving payment processing code:
   - PCI-DSS compliant?
   - Audit logging present?
   - Error handling secure?
   - Encryption used?
```
**Why**: Compliance is non-negotiable.

**5. Ask Claude to explain complex changes**
```
> Before I approve this refactoring, explain:
  1. What's changing and why
  2. How it improves the code
  3. Any potential risks
  4. How to test it
```
**Why**: Understand before approving.

**6. Use plan mode for code review assistance**
```bash
claude --permission-mode plan
> Review src/payment.ts for security issues
> Check for SQL injection vulnerabilities
```
**Why**: Get expert review without modifications.

### ❌ DON'T

**1. NEVER approve without understanding**
```
❌ [A] [A] [A]  # Blind approval
```
**Why**: May introduce bugs, security issues.

**2. NEVER skip testing after changes**
```
# After Claude makes changes
❌ git add . && git commit  # Without testing!

✅ npm test                 # Run tests first
✅ Manual testing if needed
✅ Then commit
```
**Why**: Catch bugs before they reach production.

**3. NEVER ignore linting/formatting errors**
```bash
# After changes
npm run lint
npm run format
```
**Why**: Maintains code quality standards.

**4. NEVER accept changes without tests**
```
❌ > Add feature X
   [Claude adds feature without tests]
   [You approve]

✅ > Add feature X with comprehensive tests
```
**Why**: Untested code is buggy code.

**5. NEVER skip security review**
```
For sensitive areas:
- Authentication
- Authorization
- Payment processing
- Data encryption
- API endpoints

ALWAYS extra review!
```
**Why**: Security vulnerabilities are costly.

---

## Team Collaboration

### ✅ DO

**1. Share team configuration in git**
```bash
git add .claude/settings.json
git add .claude/CLAUDE.md
git add .claude/commands/
git commit -m "chore: Add Claude Code team standards"
```
**Why**: Consistent setup across team.

**2. Document custom commands**
```markdown
## Team Commands
- `/compliance-check` - Audit code for PCI-DSS/SOX
- `/security-review` - Security vulnerability scan
- `/api-docs` - Generate API documentation
```
**Why**: Everyone knows available tools.

**3. Share best practices in CLAUDE.md**
```markdown
## Team Best Practices
1. Always start in project directory
2. Use plan mode for exploration
3. Manual git operations only
4. Review all changes before approval
```
**Why**: Consistent team behavior.

**4. Review each other's .claude configurations**
```bash
# In PR reviews, check:
- Is .claude/settings.json appropriate?
- Are hooks properly configured?
- Is CLAUDE.md up to date?
```
**Why**: Maintain standards across team.

**5. Hold team sessions to share learnings**
```
Monthly Claude Code sharing:
- Useful prompts discovered
- Custom commands created
- Common issues and solutions
```
**Why**: Continuous improvement.

### ❌ DON'T

**1. NEVER use inconsistent configurations**
**Why**: Creates confusion, different behaviors.

**2. NEVER skip documenting new commands**
**Why**: Team doesn't know they exist.

**3. NEVER create conflicting standards**
```
Bad: Different developers document different standards in CLAUDE.md
Good: Team agrees on one standard
```
**Why**: Consistency across codebase.

**4. NEVER hoard useful prompts**
**Why**: Share to help whole team.

---

## Model Usage

### ✅ DO

**1. Use Sonnet (only available model in AWS Bedrock)**
```bash
claude --model sonnet
# or just
claude  # Uses Sonnet by default
```
**Why**: Currently only model available in banking environment.

**2. Understand model capabilities**
```
Sonnet is suitable for:
- All development tasks
- Code review
- Complex problem-solving
- Quick queries
- Security analysis
```
**Why**: No need to wait for other models.

**3. Set default model in config**
```json
{
  "defaultModel": "sonnet"
}
```
**Why**: Consistency across team.

### ❌ DON'T

**1. NEVER expect Opus or Haiku (not yet available)**
```bash
# Won't work in AWS Bedrock
claude --model opus   # Not available
claude --model haiku  # Not available
```
**Why**: Only Sonnet available currently.

**2. NEVER worry about using "wrong" model**
**Why**: Sonnet handles all use cases effectively.

---

## Summary: Top 10 Critical DOs and DON'Ts

### 🔒 Top 10 DOs

1. ✅ **Review EVERY change before approval**
2. ✅ **Execute ALL git commands manually** (banking policy)
3. ✅ **Start Claude in specific project directory only**
4. ✅ **Use plan mode for unfamiliar codebases**
5. ✅ **Deny Bash tool in .claude/settings.json**
6. ✅ **Enable audit logging and secret detection hooks**
7. ✅ **Be specific and detailed in all requests**
8. ✅ **Review for PII/sensitive data before approval**
9. ✅ **Document standards in CLAUDE.md**
10. ✅ **Start fresh sessions regularly**

### 🚫 Top 10 DON'Ts

1. ❌ **NEVER use auto-approve mode**
2. ❌ **NEVER let Claude execute git commands**
3. ❌ **NEVER start Claude in home directory**
4. ❌ **NEVER paste real customer data/PII**
5. ❌ **NEVER commit secrets or credentials**
6. ❌ **NEVER skip security review of changes**
7. ❌ **NEVER use vague requests**
8. ❌ **NEVER ignore Claude's warnings**
9. ❌ **NEVER use production credentials in dev**
10. ❌ **NEVER approve without understanding**

---

## Quick Decision Tree

**Should I approve this change?**

```
┌─────────────────────────────┐
│ Did I review ALL changes?   │
└─────────┬───────────────────┘
          │
          ├── No ──→ [R] Reject, review first
          │
          ├── Yes
          │
┌─────────▼────────────────────┐
│ Any secrets/PII exposed?     │
└─────────┬────────────────────┘
          │
          ├── Yes ──→ [R] Reject, fix first
          │
          ├── No
          │
┌─────────▼────────────────────┐
│ Security requirements met?   │
└─────────┬────────────────────┘
          │
          ├── No ──→ [R] Reject, add security
          │
          ├── Yes
          │
┌─────────▼────────────────────┐
│ Tests included/passing?      │
└─────────┬────────────────────┘
          │
          ├── No ──→ Ask for tests
          │
          ├── Yes
          │
┌─────────▼────────────────────┐
│ Understand what's changing?  │
└─────────┬────────────────────┘
          │
          ├── No ──→ Ask Claude to explain
          │
          ├── Yes ──→ [A] Approve!
          │
```

---

## Resources

- **Full Documentation**: [README.md](../README.md)
- **Quick Reference**: [commands-cheatsheet.md](./commands-cheatsheet.md)
- **Git Policy**: [Section 12 - Git Integration](../05-integration/12-git-integration.md)
- **Security Guide**: [Section 9 - Security & Compliance](../04-security/09-security-compliance.md)
- **Banking Standards**: [Section 13 - Standards & Best Practices](../05-integration/13-standards-best-practices.md)

---

**Remember**: These guidelines protect you, your team, and the bank. When in doubt, err on the side of caution!

**Version:** 1.0
**Last Updated:** 2025-10-21
**Maintained By:** Banking IT - Data Chapter
