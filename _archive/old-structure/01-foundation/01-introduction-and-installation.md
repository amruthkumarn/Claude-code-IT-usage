# Section 1: Introduction & Installation

## Table of Contents
1. [What is Claude Code?](#what-is-claude-code)
2. [Why Use Claude Code in Banking IT?](#why-use-claude-code-in-banking-it)
3. [Key Features](#key-features)
4. [Prerequisites](#prerequisites)
5. [Installation](#installation)
6. [Authentication & Login](#authentication--login)
7. [Verification](#verification)
8. [Troubleshooting Installation](#troubleshooting-installation)

---

## What is Claude Code?

Claude Code is Anthropic's official agentic coding tool that brings AI-powered assistance directly to your terminal. It's designed to help developers with:

- **Building Features**: Describe what you want in plain English, and Claude will plan, write, and test the code
- **Debugging**: Analyze codebases, identify problems, and implement fixes
- **Code Navigation**: Understand project structures and pull information from documentation
- **Task Automation**: Handle repetitive tasks like fixing lint issues, resolving merge conflicts, and writing release notes

### How It Works

Claude Code operates in your terminal and can:
- Read and understand your codebase
- Edit files directly (with your approval)
- Run commands (pip, pytest, spark-submit, etc.)
- Search through code and documentation
- Integrate with external tools via MCP (Model Context Protocol)

🏦 **Banking IT Policy**: Git operations (commit, push, pull) must be executed **manually by developers**. Claude can draft commit messages and PR descriptions, but YOU execute the git commands. See [Section 12](../05-integration/12-git-integration.md) for full git workflow.

### Architecture

```
┌─────────────────────────────────────────────┐
│           User Terminal                      │
│  ┌────────────────────────────────────┐    │
│  │     Claude Code CLI (claude)       │    │
│  └─────────────┬──────────────────────┘    │
│                │                              │
│  ┌─────────────▼──────────────────────┐    │
│  │   Anthropic Claude AI Models       │    │
│  └─────────────┬──────────────────────┘    │
│                │                              │
│  ┌─────────────▼──────────────────────┐    │
│  │        Your Codebase               │    │
│  │  • Read files                       │    │
│  │  • Edit files (with approval)       │    │
│  │  • Run commands (pytest, spark-submit) │    │
│  │  • Draft git messages (manual exec) │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Model Availability (AWS Bedrock):**
- ✅ **Sonnet** - Currently available (recommended for banking IT use)
- ⏳ **Opus** - In progress
- ❌ **Haiku** - Not currently available

---

## Why Use Claude Code in Banking IT?

### For Individual Developers
- **Faster Development**: Reduce time spent on boilerplate code and repetitive tasks
- **Code Quality**: Get instant code reviews and suggestions
- **Learning Tool**: Understand unfamiliar codebases quickly
- **Documentation**: Generate and maintain technical documentation

### For Data Chapter Teams
- **Consistency**: Enforce coding standards across teams
- **Knowledge Sharing**: Embed team conventions in project configuration
- **Onboarding**: Help new developers understand complex banking systems
- **Compliance**: Automate compliance checks and security reviews

### Banking-Specific Benefits
- **Security First**: Read-only by default, requires explicit approval for changes
- **Audit Trail**: All actions can be logged and monitored
- **Regulatory Compliance**: Integrate compliance checks into workflows
- **Code Review**: Automated initial review before human review
- **Legacy Code**: Understand and modernize legacy banking systems

---

## Key Features

### 1. Interactive REPL (Read-Eval-Print Loop)
Work conversationally with Claude in your terminal, maintaining context throughout the session.

### 2. Direct File Access
Claude can read, search, and edit files in your working directory (with your approval).

### 3. Command Execution
Run shell commands, tests, and build processes (pip, pytest, spark-submit, etc.).

**Note**: Git operations must be executed manually per banking IT policy.

### 4. Plan Mode
Explore code safely without making any changes - perfect for understanding new codebases.

### 5. Customizable
- Custom slash commands for frequent tasks
- Memory system to remember preferences and standards
- Hooks for automation and policy enforcement
- Sub-agents for specialized tasks

### 6. Enterprise Security
- Permission-based access control
- Prompt injection protection
- Network request approval
- Secure credential storage

### 7. Version Control Integration
Create commits, pull requests, and manage branches with AI assistance.

### 8. Extensibility (MCP)
Connect to external tools: databases, monitoring systems, issue trackers, and internal APIs.

---

## Prerequisites

Before installing Claude Code, ensure you have:

### Required

1. **Operating System**
   - Windows 10/11 (with WSL2 recommended)

2. **Terminal/Command Prompt**
   - Windows: PowerShell (recommended), Command Prompt, or WSL2

3. **Claude Account**
   - Claude.ai account (for individual use) OR
   - Claude Console account (for enterprise/team use)
   - API access enabled

4. **For pip Installation**
   - Python 3.8 or later
   - pip (comes with Python)

### Recommended

- **Git** (for version control features)
- **Code editor** (VS Code, IntelliJ, etc.) for viewing changes
- **Basic terminal knowledge** (navigating directories, running commands)

### Banking IT Considerations

- **Network Access**: Ensure access to `https://api.anthropic.com`
- **Proxy Configuration**: May need corporate proxy settings
- **Firewall**: Port 443 (HTTPS) must be accessible
- **SSO/Authentication**: Check with IT for enterprise authentication requirements
- **Python Environment**: Consider using virtual environments for isolation

---

## Installation

Claude Code can be installed in two ways. Choose the method that works best for your environment.

### Method 1: pip (Python Package Manager)

**Best for:** Developers who already have Python installed

#### Prerequisites
```bash
# Check if Python is installed (need 3.8+)
python --version

# Check if pip is installed
pip --version
```

#### Installation Steps

```bash
# Install globally
pip install anthropic-claude-code

# Verify installation
claude --version
```

**Banking IT Note**: If your organization uses a private PyPI registry, you may need to configure pip:

```bash
# Set PyPI index (if required)
pip config set global.index-url https://your-internal-pypi.bank.com/simple/

# Or install with specific index
pip install anthropic-claude-code --index-url https://pypi.org/simple/
```

#### Behind Corporate Proxy

```bash
# Set proxy for pip
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080

# Then install
pip install anthropic-claude-code
```

---

### Method 2: Native Install (Windows)

**Option A: PowerShell (Native)**
```powershell
# Run in PowerShell as Administrator
irm https://install.claude.ai/claude-code/windows | iex
```

**Option B: WSL2 (Recommended for Banking IT)**
```bash
# Inside WSL2 Ubuntu/Debian
curl -fsSL https://install.claude.ai/claude-code | sh
```

**Banking IT Note**: WSL2 provides better isolation and is recommended for banking environments.

---

## Authentication & Login

After installation, you need to authenticate Claude Code with your Claude account.

### First-Time Login

```bash
# Start the login process
claude /login
```

This will:
1. Open a browser window to authenticate
2. Redirect to Claude login page
3. Generate an authentication token
4. Store credentials securely in your system keychain

### Login Flow

```
┌──────────────────────────────────────────────┐
│  Terminal: claude /login                     │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  Browser opens: https://claude.ai/login      │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  User enters credentials                      │
│  (or uses SSO if configured)                  │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  Authorization granted                        │
│  Token stored in system keychain              │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  Ready to use: claude                         │
└──────────────────────────────────────────────┘
```

### Alternative: API Key Authentication

For enterprise users or CI/CD environments:

**PowerShell:**
```powershell
# Set API key as environment variable (current session)
$env:ANTHROPIC_API_KEY = "your-api-key-here"

# Or add to PowerShell profile (persistent)
Add-Content $PROFILE 'Set-Item -Path Env:ANTHROPIC_API_KEY -Value "your-api-key-here"'
```

**WSL2/Linux:**
```bash
# Set API key as environment variable
export ANTHROPIC_API_KEY="your-api-key-here"

# Or add to shell profile
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.bashrc
```

### Enterprise SSO (Banking IT)

If your organization uses Single Sign-On:

1. Contact your IT administrator for the enterprise Claude Console URL
2. Use the provided enterprise login endpoint
3. Follow your organization's SSO flow (SAML, OIDC, etc.)

```bash
# Enterprise login (if configured)
claude /login --enterprise
```

---

## Verification

After installation and login, verify Claude Code is working correctly.

### Basic Verification

```bash
# Check version
claude --version

# Expected output:
# claude-code version 1.x.x

# Check authentication status
claude /login

# Expected output if logged in:
# Already logged in as: your-email@bank.com
```

### Test First Session

```bash
# Navigate to a project directory
cd ~/projects/my-test-project

# Start Claude Code
claude

# You should see:
# Welcome to Claude Code!
# You're in: /Users/yourname/projects/my-test-project
#
# Type /help for available commands
# Press ? for keyboard shortcuts
```

### Quick Test Query

In the Claude Code REPL:

```
> What files are in this directory?

# Claude will use the appropriate tools to list files
# and provide a summary
```

### Exit Claude Code

```bash
# In the REPL, press:
Ctrl+D (or type /exit)
```

---

## Troubleshooting Installation

### Issue: "command not found: claude"

**Solution (PowerShell):**

```powershell
# Check if Claude is in PATH
Get-Command claude

# If not found, add Python Scripts path to PATH (pip installation)
$pythonPath = python -c "import site; print(site.USER_BASE)"
$env:Path += ";$pythonPath\Scripts"

# Add to PowerShell profile for persistence
Add-Content $PROFILE "`$env:Path += `";$(python -c 'import site; print(site.USER_BASE)')\Scripts`""
```

**Solution (WSL2):**

```bash
# Check if Claude is in PATH
which claude

# If not found, add to PATH (pip installation)
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
source ~/.bashrc
```

### Issue: "Python version not supported"

**Solution:**

```bash
# Check Python version
python --version

# If < 3.8, update Python
# Using pyenv (Python Version Manager)
pyenv install 3.11
pyenv global 3.11

# Or download from: https://www.python.org/downloads/
```

### Issue: "Permission denied" during pip install

**Solution (PowerShell - Run as Administrator):**

```powershell
# Run PowerShell as Administrator, then install
pip install anthropic-claude-code
```

**Solution (WSL2):**

```bash
# Option 1: Install for current user only (recommended)
pip install --user anthropic-claude-code

# Option 2: Use sudo (system-wide installation)
sudo pip install anthropic-claude-code

# Option 3: Use virtual environment (best practice)
python -m venv ~/.venv/claude
source ~/.venv/claude/bin/activate
pip install anthropic-claude-code
```

### Issue: "Cannot connect to api.anthropic.com"

**Banking IT Solutions (PowerShell):**

```powershell
# Check network connectivity
curl -I https://api.anthropic.com

# If behind corporate proxy, set proxy
$env:HTTP_PROXY = "http://proxy.bank.com:8080"
$env:HTTPS_PROXY = "http://proxy.bank.com:8080"

# Add to PowerShell profile for persistence
Add-Content $PROFILE 'Set-Item -Path Env:HTTP_PROXY -Value "http://proxy.bank.com:8080"'
Add-Content $PROFILE 'Set-Item -Path Env:HTTPS_PROXY -Value "http://proxy.bank.com:8080"'
```

**Banking IT Solutions (WSL2):**

```bash
# Check network connectivity
curl -I https://api.anthropic.com

# If behind corporate proxy, set proxy
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080

# Add to shell profile for persistence
echo 'export HTTP_PROXY=http://proxy.bank.com:8080' >> ~/.bashrc
echo 'export HTTPS_PROXY=http://proxy.bank.com:8080' >> ~/.bashrc
```

### Issue: "Authentication failed"

**Solution:**

```bash
# Clear existing credentials
rm ~/.claude/credentials.json

# Login again
claude /login

# If using API key, verify it's correct
echo $ANTHROPIC_API_KEY
```

### Issue: SSL/TLS Certificate Errors

**Banking IT Solution (PowerShell):**

```powershell
# If using corporate SSL inspection
# Set custom CA certificate
$env:NODE_EXTRA_CA_CERTS = "C:\path\to\corporate-ca-bundle.crt"

# Add to PowerShell profile
Add-Content $PROFILE 'Set-Item -Path Env:NODE_EXTRA_CA_CERTS -Value "C:\path\to\corporate-ca-bundle.crt"'
```

**Banking IT Solution (WSL2):**

```bash
# If using corporate SSL inspection
# Set custom CA certificate
export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca-bundle.crt

# Add to shell profile
echo 'export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca-bundle.crt' >> ~/.bashrc
```

### Issue: Windows Installation Fails

**Solution:**

```bash
# Use WSL2 instead (recommended)
# 1. Enable WSL2:
wsl --install

# 2. Install Ubuntu from Microsoft Store
# 3. Inside WSL2, follow Linux installation steps
curl -fsSL https://install.claude.ai/claude-code | sh
```

---

## Next Steps

Now that Claude Code is installed and authenticated:

1. **[Continue to Section 2: Core Concepts](./02-core-concepts.md)** - Understand how Claude Code works
2. **[Jump to Section 3: Getting Started](../02-basics/03-getting-started.md)** - Start using Claude Code
3. **[Review Quick Reference](../quick-reference/commands-cheatsheet.md)** - Common commands and shortcuts

---

## Summary

In this section, you learned:

- What Claude Code is and its key features
- Why Claude Code is valuable for banking IT teams
- Prerequisites for installation
- Two installation methods (pip, native Windows)
- Authentication and login process
- How to verify installation
- Common troubleshooting solutions

**Key Takeaways:**

1. Claude Code brings AI assistance directly to your terminal
2. It's security-first with read-only defaults and approval workflows
3. Multiple installation methods support different environments
4. Enterprise authentication integrates with existing SSO
5. Banking IT environments may require proxy and certificate configuration

---
