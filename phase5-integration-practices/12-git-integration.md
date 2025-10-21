# Section 12: Version Control Integration

## Table of Contents
1. [Git with Claude Code](#git-with-claude-code)
2. [Creating Commits](#creating-commits)
3. [Creating Pull Requests](#creating-pull-requests)
4. [Best Practices](#best-practices)

---

## Git with Claude Code

Claude Code integrates seamlessly with Git for version control.

### What Claude Can Do

- View git status and diffs
- Create commits with descriptive messages
- Create branches
- Generate pull requests
- Resolve merge conflicts
- Generate release notes from commit history

### What You Control

- Pushing to remote (Claude asks for approval)
- Merging pull requests
- Git configuration
- Branch protection rules

---

## Creating Commits

### Basic Commit Workflow

```
> Add feature X

Claude: [makes code changes]

> Create a commit for these changes

Claude will:
1. Run git status
2. Run git diff to see changes
3. Review commit history for style
4. Draft a commit message
5. Ask for approval
6. Run git add and git commit
```

### Commit Message Format

Claude follows your project's commit message conventions:

**Conventional Commits:**
```
feat(auth): add password reset functionality

- Implement password reset email
- Add reset token generation
- Create reset form UI

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

**Simple Format:**
```
Add password reset feature

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

### Customizing Commit Format

Add to `CLAUDE.md`:
```markdown
## Git Commit Standards

Use Conventional Commits format:
- feat: New feature
- fix: Bug fix
- refactor: Code refactoring
- test: Adding tests
- docs: Documentation
- chore: Maintenance

Example:
```
feat(payments): add Stripe integration

- Implement payment processing
- Add webhook handling
- Create invoice generation

Closes #123
```
```

### Banking Example

```
> Implement transaction validation

[Claude makes changes]

> Commit these changes

Claude creates:
```
feat(transactions): add balance validation

- Validate sufficient funds before transfer
- Add overdraft limit checking
- Implement fraud detection rules
- Add audit logging for failed transactions

Compliance: Implements SOX requirement 4.2.1

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```
```

---

## Creating Pull Requests

### PR Creation Workflow

```
> Create a pull request for this feature

Claude will:
1. Check current branch
2. Run git diff main...HEAD (or base branch)
3. Review ALL commits in branch
4. Generate comprehensive PR description
5. Push branch to remote (with approval)
6. Create PR using gh CLI
```

### PR Description Format

```markdown
## Summary
- Implemented password reset functionality
- Added email notification system
- Created user-facing reset form

## Changes
- `src/auth/reset.ts`: Password reset logic
- `src/email/templates/reset.html`: Email template
- `tests/auth/reset.test.ts`: Unit tests

## Testing
- [x] Unit tests passing
- [x] Integration tests passing
- [x] Manual testing completed
- [x] Security review completed

## Compliance
- Passwords hashed with bcrypt (cost: 12)
- Reset tokens expire after 1 hour
- Email sent over encrypted connection
- Audit logging implemented

🤖 Generated with Claude Code
```

### Using GitHub CLI

```bash
# Install gh CLI
brew install gh

# Authenticate
gh auth login

# Create PR with Claude
> Create a PR for this feature
```

Claude executes:
```bash
gh pr create \
  --title "feat: Add password reset functionality" \
  --body "$(cat <<'EOF'
## Summary
...
EOF
)"
```

---

## Best Practices

### 1. Work in Feature Branches

```bash
# Create branch
git checkout -b feature/password-reset

# Work with Claude
claude

> Implement password reset
> Run tests
> Commit changes

# Create PR
> Create a pull request

# Merge via GitHub/GitLab UI (not Claude)
```

### 2. Review Before Committing

```
> Show me what changes will be committed

Claude runs:
git diff --staged

Review carefully before approving commit!
```

### 3. Atomic Commits

```
# Good: One feature per commit
> Commit the password reset feature

# Bad: Multiple unrelated changes
> Commit everything
```

### 4. Never Auto-Push to Main

Add to `CLAUDE.md`:
```markdown
## Git Rules

NEVER push directly to:
- main
- master
- production
- release/*

Always use feature branches and pull requests.
```

### 5. Pre-Commit Hooks

Use hooks to enforce standards:

`.claude/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash.*git commit",
        "hooks": [{
          "type": "command",
          "command": "npm run lint && npm test",
          "description": "Run tests before commit"
        }]
      }
    ]
  }
}
```

---

## Common Workflows

### Workflow 1: Feature Development

```bash
1. Create branch:
   git checkout -b feature/add-2fa

2. Develop with Claude:
   > Implement 2FA using TOTP
   > Add tests
   > Update documentation

3. Commit:
   > Create a commit for the 2FA feature

4. Push and PR:
   > Create a pull request
```

### Workflow 2: Bug Fix

```bash
1. Create branch:
   git checkout -b fix/login-timeout

2. Fix with Claude:
   > Fix the login timeout issue in src/auth/session.ts

3. Commit:
   > Commit the bug fix

4. Push and PR:
   > Create a pull request with the fix
```

### Workflow 3: Merge Conflict Resolution

```
> I have a merge conflict in src/payment.ts. Help me resolve it.

Claude will:
1. Read the conflicted file
2. Show both versions
3. Suggest resolution
4. Make the changes
5. Help you commit the resolution
```

---

## Summary

In this section, you learned:

### Core Concepts
- Git integration for commits and PRs
- Claude follows your commit conventions
- Approval required for push operations

### Implementation
- Creating commits with Claude
- Generating pull requests
- Using GitHub CLI integration
- Customizing commit messages

### Best Practices
- Feature branch workflow
- Atomic commits
- Pre-commit validation
- Never push directly to main

---

## Next Steps

1. **[Continue to Section 13: Standards & Best Practices](./13-standards-best-practices.md)** - Team standards
2. **Try creating a commit** - Make a change and let Claude commit it

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-11
