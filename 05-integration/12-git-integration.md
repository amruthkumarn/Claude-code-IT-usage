# Section 12: Version Control Integration

## Table of Contents
1. [Git with Claude Code](#git-with-claude-code)
2. [Banking IT Policy: Manual Git Operations](#banking-it-policy-manual-git-operations)
3. [Claude's Role in Git Workflow](#claudes-role-in-git-workflow)
4. [Manual Commit Workflow](#manual-commit-workflow)
5. [Manual Pull Request Workflow](#manual-pull-request-workflow)
6. [Best Practices](#best-practices)

---

## Git with Claude Code

### 🏦 Banking IT Policy

**IMPORTANT: Git automation via Claude Code is NOT approved for banking environments.**

All git operations (commit, push, pull, merge) MUST be performed manually by developers. This section documents how to work with Claude Code while maintaining manual control of all version control operations.

---

## Banking IT Policy: Manual Git Operations

### What is NOT Allowed

Claude Code must **NEVER** execute the following commands:
- ❌ `git add`
- ❌ `git commit`
- ❌ `git push`
- ❌ `git pull`
- ❌ `git merge`
- ❌ `git checkout`
- ❌ `git branch`
- ❌ `gh pr create`
- ❌ Any other git write operations

### Why Manual Git Operations?

Banking IT requirements:
1. **Audit Trail**: All git operations must be traceable to individual developers
2. **Approval Workflow**: Code changes require manual review before commit
3. **Compliance**: SOX, PCI-DSS regulations require human oversight
4. **Security**: Prevent accidental commits of sensitive data
5. **Accountability**: Developers are personally responsible for commits

### Enforcement

Configure `.claude/settings.json` to prevent git automation:

```json
{
  "permissions": {
    "deny": ["Bash"]
  }
}
```

This prevents Claude from executing any shell commands, including git operations.

---

## Claude's Role in Git Workflow

### What Claude CAN Do (Read-Only)

Claude can help you **understand** your git state:
- ✅ **Suggest** commit messages (you copy and use manually)
- ✅ **Generate** PR descriptions (you copy to GitHub/GitLab)
- ✅ **Explain** git diff output (if you paste it)
- ✅ **Draft** release notes from commit history (if you paste it)
- ✅ **Help** resolve merge conflicts (you make final changes)

### What You MUST Do Manually

All git commands are your responsibility:
1. Review changes with `git status` and `git diff`
2. Stage changes with `git add`
3. Create commits with `git commit`
4. Push to remote with `git push`
5. Create PRs via GitHub/GitLab UI or `gh` CLI manually

---

## Manual Commit Workflow

### Step 1: Development with Claude

```bash
# Work with Claude
claude

> Implement password reset functionality
```

Claude makes code changes (with your approval).

### Step 2: Review Changes Manually

```bash
# Exit Claude
Ctrl+D

# Review what changed
git status
git diff

# Review specific files
git diff src/auth/reset.ts
```

### Step 3: Ask Claude for Commit Message

```bash
# Start Claude in read-only mode
claude --permission-mode plan

> I made the following changes:
> [paste git diff or describe changes]
>
> Please draft a commit message following Conventional Commits format
> for our banking project
```

Claude suggests:
```
feat(auth): add password reset functionality

- Implement password reset email workflow
- Add secure token generation (expires 1 hour)
- Create password reset form with validation
- Add audit logging for reset attempts

Compliance: Implements security requirement SEC-4.2
```

### Step 4: Create Commit Manually

```bash
# Exit Claude
Ctrl+D

# Stage changes
git add src/auth/reset.ts src/email/templates/reset.html tests/auth/reset.test.ts

# Commit with message (copy from Claude's suggestion)
git commit -m "feat(auth): add password reset functionality

- Implement password reset email workflow
- Add secure token generation (expires 1 hour)
- Create password reset form with validation
- Add audit logging for reset attempts

Compliance: Implements security requirement SEC-4.2"
```

### Step 5: Push Manually

```bash
# Push to remote
git push origin feature/password-reset
```

---

## Manual Pull Request Workflow

### Step 1: Prepare PR

```bash
# Ensure all changes are committed
git status

# Check branch status
git log --oneline origin/main..HEAD
```

### Step 2: Ask Claude for PR Description

```bash
claude --permission-mode plan

> I have the following commits on my branch:
> [paste git log output]
>
> Please generate a comprehensive pull request description
> for our banking IT code review process
```

Claude generates:
```markdown
## Summary
Implementation of password reset functionality with security and compliance features.

## Changes
- **src/auth/reset.ts**: Password reset logic with secure token generation
- **src/email/templates/reset.html**: Email template for reset links
- **tests/auth/reset.test.ts**: Comprehensive unit tests (95% coverage)
- **docs/api/auth.md**: API documentation update

## Security Considerations
- Reset tokens expire after 1 hour
- Tokens are cryptographically secure (32 bytes random)
- Rate limiting: 3 attempts per hour per email
- Passwords hashed with bcrypt (cost factor: 12)
- Email sent over TLS 1.3

## Compliance
- **SOX**: Audit trail for all password resets (requirement 4.2.1)
- **PCI-DSS**: Secure authentication mechanisms (requirement 8.2)
- **GDPR**: User consent for email notification (requirement 7.2)

## Testing Completed
- [x] Unit tests passing (127 tests, 95% coverage)
- [x] Integration tests passing
- [x] Manual QA testing completed
- [x] Security review by security team
- [x] Performance testing (handles 1000 requests/min)

## Deployment Notes
- No database migrations required
- New environment variable: `PASSWORD_RESET_TOKEN_EXPIRY` (default: 3600)
- Email template requires approval from compliance team

## Reviewers
@security-team @compliance-team
```

### Step 3: Create PR Manually

**Option A: GitHub Web UI**
1. Go to GitHub repository
2. Click "New Pull Request"
3. Select your branch
4. Copy Claude's description into PR body
5. Add reviewers
6. Create PR

**Option B: GitHub CLI (Manual)**
```bash
# Create PR manually using gh CLI
gh pr create \
  --title "feat(auth): Add password reset functionality" \
  --body-file pr-description.md \
  --reviewer @security-team,@compliance-team

# Where pr-description.md contains Claude's generated description
```

---

## Best Practices

### 1. Always Work in Feature Branches

```bash
# Create branch manually
git checkout -b feature/transaction-validation

# Develop with Claude
claude
> Implement transaction validation logic

# Commit manually (as described above)
git add .
git commit -m "feat(transactions): add validation logic"

# Push manually
git push origin feature/transaction-validation
```

### 2. Review Every Change Before Committing

```bash
# Always review
git status
git diff

# For specific concerns, ask Claude
claude --permission-mode plan
> Review the changes in src/payment.ts for security issues
```

### 3. Use Atomic Commits

```bash
# Good: Separate commits for separate concerns
git add src/auth/
git commit -m "feat(auth): add 2FA support"

git add tests/auth/
git commit -m "test(auth): add 2FA tests"

# Bad: Everything in one commit
git add .
git commit -m "add 2FA"
```

### 4. Leverage Claude for Commit Message Quality

```bash
# Before each commit, ask Claude
claude --permission-mode plan

> I changed the following files:
> - src/payments/processor.ts: Add retry logic
> - src/payments/validator.ts: Add amount validation
>
> Draft a commit message following our Conventional Commits standard
```

### 5. Document Git Standards in CLAUDE.md

`.claude/CLAUDE.md`:
```markdown
## Git Commit Standards

### Format
Use Conventional Commits format:
- `feat(scope)`: New feature
- `fix(scope)`: Bug fix
- `refactor(scope)`: Code refactoring
- `test(scope)`: Adding tests
- `docs(scope)`: Documentation
- `chore(scope)`: Maintenance

### Example
```
feat(payments): add Stripe webhook handling

- Implement webhook endpoint
- Add signature verification
- Handle payment success/failure events
- Add error logging

Compliance: PCI-DSS requirement 12.8.2
```

### Scopes
- auth: Authentication
- payments: Payment processing
- transactions: Transaction handling
- audit: Audit logging
- compliance: Compliance features
```

### 6. Use Pre-Commit Hooks (Manual)

Set up git hooks to run before commit:

`.git/hooks/pre-commit`:
```bash
#!/bin/bash
# Run linting
npm run lint || exit 1

# Run tests
npm test || exit 1

# Check for secrets
./scripts/detect-secrets.sh || exit 1
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

---

## Common Workflows

### Workflow 1: Feature Development

```bash
# 1. Create branch manually
git checkout -b feature/add-2fa

# 2. Develop with Claude
claude
> Implement 2FA using TOTP
> Add tests for 2FA

# 3. Exit and review
Ctrl+D
git status
git diff

# 4. Get commit message from Claude
claude --permission-mode plan
> Draft commit message for 2FA implementation

# 5. Commit manually
git add .
git commit -m "[paste Claude's suggested message]"

# 6. Push manually
git push origin feature/add-2fa

# 7. Get PR description from Claude
claude --permission-mode plan
> Generate PR description for 2FA feature

# 8. Create PR manually via GitHub UI
```

### Workflow 2: Bug Fix

```bash
# 1. Create branch manually
git checkout -b fix/login-timeout

# 2. Fix with Claude
claude
> Fix the session timeout issue in src/auth/session.ts

# 3. Review and commit manually
Ctrl+D
git diff
git add src/auth/session.ts
git commit -m "fix(auth): prevent session timeout on active users"

# 4. Push manually
git push origin fix/login-timeout

# 5. Create PR manually
```

### Workflow 3: Code Review Assistance

```bash
# You receive PR review comments
# Use Claude to help address them

claude
> I received this review comment on PR #123:
> "The error handling in payment.ts could be more specific"
>
> Please improve the error handling in src/payments/processor.ts

# Claude makes changes

# Review and commit manually
Ctrl+D
git add src/payments/processor.ts
git commit -m "refactor(payments): improve error handling specificity"
git push
```

---

## Configuration for Banking IT

### Recommended Settings

`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Grep", "Glob"],
    "deny": ["Bash"]
  },

  "defaultModel": "sonnet",

  "env": {
    "GIT_OPERATIONS": "MANUAL_ONLY"
  }
}
```

### Project Memory

`.claude/CLAUDE.md`:
```markdown
# Git Operations Policy

## CRITICAL: Manual Git Only

ALL git operations MUST be performed manually by developers.

When asked to commit or push:
1. Suggest commit message
2. Suggest git commands
3. NEVER execute git commands
4. Remind user to perform git operations manually

## Workflow
1. Claude makes code changes (with approval)
2. Developer reviews changes manually (git diff)
3. Developer asks Claude for commit message
4. Developer executes git commands manually
5. Developer creates PR manually (can use Claude's description)
```

---

## Summary

### Banking IT Git Policy
- ✅ Claude helps write code
- ✅ Claude suggests commit messages
- ✅ Claude generates PR descriptions
- ❌ Claude NEVER executes git commands
- ✅ All git operations performed manually by developers

### Developer Workflow
1. **Develop**: Work with Claude to write/modify code
2. **Review**: Manually review changes with `git diff`
3. **Draft**: Ask Claude for commit message/PR description
4. **Commit**: Manually execute git commands
5. **Push**: Manually push to remote
6. **PR**: Manually create pull request

### Key Benefits
- Maintains audit trail and accountability
- Complies with banking regulations
- Prevents accidental commits
- Developer retains full control
- Leverages Claude for quality commit messages

---

## Next Steps

1. **[Continue to Section 13: Standards & Best Practices](./13-standards-best-practices.md)** - Team standards
2. **Configure your project** - Set up `.claude/settings.json` to deny Bash
3. **Update CLAUDE.md** - Document manual git policy

---

**Document Version:** 1.1
**Last Updated:** 2025-10-21
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-11
**Policy Status:** Manual Git Operations REQUIRED
