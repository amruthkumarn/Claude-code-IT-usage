# Claude Code - Quick Reference Card

## Installation

```bash
# npm
npm install -g @anthropic-ai/claude-code

# Homebrew (macOS/Linux)
brew install --cask claude-code

# Verify
claude --version
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `claude` | Start interactive session |
| `claude "query"` | Start with initial prompt |
| `claude -p "query"` | Print mode (non-interactive) |
| `claude --version` | Show version |
| `claude /login` | Login/check auth |
| `claude --continue` | Resume last session |

## Common Flags

| Flag | Description |
|------|-------------|
| `--model sonnet` | Select AI model (Sonnet only in AWS Bedrock) |
| `--permission-mode plan` | Read-only mode |
| `--permission-mode auto-approve` | Auto-approve (dangerous!) |
| `--add-dir <path>` | Add directory to scope |
| `--verbose` | Verbose output |

## Built-in Slash Commands

| Command | Description |
|---------|-------------|
| `/help` | Show help |
| `/clear` | Clear conversation |
| `/exit` | Exit Claude Code |
| `/model <model>` | Change model |
| `/config` | Open settings |
| `/memory` | Edit memory files |
| `/review` | Code review |
| `/output-style <style>` | Change output style |

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+D` | Exit |
| `Ctrl+C` | Cancel operation |
| `Ctrl+L` | Clear screen |
| `↑` / `↓` | Command history |
| `?` | Show shortcuts |

## Approval Prompt Keys

| Key | Action |
|-----|--------|
| `A` | Approve |
| `R` | Reject |
| `E` | Edit before applying |
| `V` | View full context |

## File Locations

| File | Purpose |
|------|---------|
| `~/.claude/settings.json` | User settings |
| `.claude/settings.json` | Project settings (commit) |
| `.claude/settings.local.json` | Personal settings (gitignore) |
| `~/.claude/CLAUDE.md` | User memory |
| `.claude/CLAUDE.md` | Project memory |
| `.claude/commands/` | Custom slash commands |
| `.claude/hooks/` | Hook scripts |

## Common Workflows

**Explore codebase:**
```bash
claude --permission-mode plan
> What is this project about?
> Explain the authentication system
```

**Fix a bug:**
```bash
claude
> The login endpoint returns 500. Can you fix it?
[Review changes]
[Approve]
```

**🏦 Banking IT: Git Operations (Manual Only):**
```bash
# 1. Make changes with Claude
claude
> Fix the login bug

# 2. Exit and review
Ctrl+D
git diff

# 3. Ask Claude for commit message
claude --permission-mode plan
> Draft commit message for login bug fix

# 4. Commit MANUALLY
git add .
git commit -m "[paste Claude's message]"

# 5. Push MANUALLY
git push

# NOTE: Git automation NOT approved for banking
```

## Permission Modes

| Mode | Read | Write | Execute |
|------|------|-------|---------|
| `interactive` (default) | ✓ | Ask | Ask |
| `auto-approve` | ✓ | ✓ | ✓ |
| `plan` | ✓ | ✗ | ✗ |
| `deny` | ✓ | ✗ | ✗ |

## Models

**AWS Bedrock Availability:**

| Model | Status | Use For |
|-------|--------|---------|
| **Sonnet** | ✅ Available | All development tasks (default) |
| **Opus** | ⏳ In Progress | Not yet available |
| **Haiku** | ❌ Not Available | Not available |

**Note:** Sonnet is currently the only model available in AWS Bedrock and is suitable for all development tasks including quick queries, general development, and complex problem-solving.

## Troubleshooting

**"command not found":**
```bash
echo 'export PATH="$PATH:$(npm bin -g)"' >> ~/.zshrc
source ~/.zshrc
```

**Authentication failed:**
```bash
claude /login
# or
export ANTHROPIC_API_KEY="sk-ant-..."
```

**Behind corporate proxy:**
```bash
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
```

**SSL certificate error:**
```bash
export NODE_EXTRA_CA_CERTS="/path/to/ca.crt"
```

## Quick Setup

```bash
# Create .claude directory
mkdir -p .claude/{commands,hooks}

# Create settings
cat > .claude/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write"]
  },
  "defaultModel": "sonnet"
}
EOF

# Create memory
echo "# Project Standards" > .claude/CLAUDE.md

# Update .gitignore
echo ".claude/settings.local.json" >> .gitignore
```

## Prompt Engineering Tips

### Be Specific
```
❌ Fix the bug
✅ The login endpoint returns 500 when password is missing.
   Add validation and return 400 with clear error message.
```

### Provide Context
```
✅ Add JWT authentication to API endpoints.
   Use existing UserService for validation.
   Token expiry: 15 minutes.
   Follow pattern in src/auth/middleware.ts
```

### Request Format
```
✅ Scan for secrets and return JSON:
   { "findings": [{"file": "...", "line": 42, "type": "api_key"}] }
```

### Chain of Thought
```
✅ Debug the transaction rollback issue.
   Think step by step:
   1. Identify where transactions start
   2. Trace error handling
   3. Check rollback calls
   4. Suggest fix
```

## 🏦 Banking IT: Dos and Don'ts (Quick Reference)

**See [Full Dos and Don'ts Guide](./dos-and-donts.md) for detailed explanations**

### Top 10 Critical DOs

1. ✅ **Review EVERY change before approval** - No blind approvals
2. ✅ **Execute ALL git commands manually** - Banking policy, no automation
3. ✅ **Start in specific project directory** - Never in ~/ (home)
4. ✅ **Use plan mode for exploration** - `--permission-mode plan`
5. ✅ **Deny Bash tool** - In `.claude/settings.json`
6. ✅ **Enable audit logging** - Use hooks for compliance
7. ✅ **Be specific in requests** - Detail > vague
8. ✅ **Review for PII/secrets** - Before every approval
9. ✅ **Document standards** - In `CLAUDE.md`
10. ✅ **Start fresh sessions regularly** - Every 1-2 hours

### Top 10 Critical DON'Ts

1. ❌ **NEVER use auto-approve** - Violates compliance
2. ❌ **NEVER let Claude run git** - Manual only!
3. ❌ **NEVER start in home directory** - `cd ~/project` first
4. ❌ **NEVER paste customer data/PII** - Use synthetic data
5. ❌ **NEVER commit secrets** - Use environment variables
6. ❌ **NEVER skip security review** - Always check changes
7. ❌ **NEVER use vague requests** - Be specific
8. ❌ **NEVER ignore warnings** - Claude's warnings are real
9. ❌ **NEVER use production creds** - Keep separate
10. ❌ **NEVER approve without understanding** - Ask questions

### Quick Security Checklist

Before approving ANY change:
```
□ Did I review all changes?
□ No secrets/API keys exposed?
□ No customer data/PII?
□ Security requirements met?
□ Tests included/passing?
□ Do I understand the change?
```

### Git Operations (Manual Only)

```bash
# 1. Code with Claude
claude
> Fix the bug

# 2. Exit and review
Ctrl+D
git diff

# 3. Get commit message
claude --permission-mode plan
> Draft commit message

# 4. Execute manually
git add .
git commit -m "[paste message]"
git push
```

### Emergency: What to Do If...

**Accidentally approved something wrong:**
```bash
# Immediately reject next approval or Ctrl+C
# Review git diff
# If committed: git reset HEAD~1
```

**Claude seems slow:**
```bash
Ctrl+D  # Exit
claude  # Start fresh
```

**Unsure about a change:**
```bash
[R]  # Reject
> Explain what this change does and why
```

**See [Full Guide](./dos-and-donts.md) for 60+ detailed guidelines**

---

## Documentation

- **Official Docs**: https://docs.claude.com/en/docs/claude-code/overview
- **Quickstart**: https://docs.claude.com/en/docs/claude-code/quickstart
- **CLI Reference**: https://docs.claude.com/en/docs/claude-code/cli-reference
- **Prompt Engineering**: https://github.com/anthropics/prompt-eng-interactive-tutorial
- **Dos and Don'ts**: [Full Guide](./dos-and-donts.md)

---

**Version:** 1.1
**Last Updated:** 2025-10-21
