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
| `--model sonnet\|opus\|haiku` | Select AI model |
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

**Create commit:**
```bash
claude
> Create a commit for these changes
[Review commit message]
[Approve]
```

**Create PR:**
```bash
claude
> Create a pull request
[Review PR description]
[Approve]
```

## Permission Modes

| Mode | Read | Write | Execute |
|------|------|-------|---------|
| `interactive` (default) | ✓ | Ask | Ask |
| `auto-approve` | ✓ | ✓ | ✓ |
| `plan` | ✓ | ✗ | ✗ |
| `deny` | ✓ | ✗ | ✗ |

## Models

| Model | Speed | Capability | Cost | Use For |
|-------|-------|------------|------|---------|
| **Sonnet** | Fast | High | Medium | General dev (default) |
| **Opus** | Slow | Highest | High | Complex problems |
| **Haiku** | Fastest | Good | Low | Quick queries |

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

## Documentation

- **Official Docs**: https://docs.claude.com/en/docs/claude-code/overview
- **Quickstart**: https://docs.claude.com/en/docs/claude-code/quickstart
- **CLI Reference**: https://docs.claude.com/en/docs/claude-code/cli-reference

---

**Version:** 1.0
**Last Updated:** 2025-10-19
