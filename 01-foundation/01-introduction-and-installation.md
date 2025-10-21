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
- Run commands (npm, testing frameworks, etc.)
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
│  │  • Run commands (npm, tests)        │    │
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
Run shell commands, tests, and build processes (npm, pytest, etc.).

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
   - macOS (10.15 or later)
   - Linux (Ubuntu 20.04+, Debian, Fedora, RHEL, etc.)
   - Windows 10/11 (with WSL2 recommended)

2. **Terminal/Command Prompt**
   - macOS: Terminal, iTerm2, or any terminal emulator
   - Linux: bash, zsh, or compatible shell
   - Windows: PowerShell, Command Prompt, or WSL2

3. **Claude Account**
   - Claude.ai account (for individual use) OR
   - Claude Console account (for enterprise/team use)
   - API access enabled

4. **For npm Installation**
   - Node.js 18.0 or later
   - npm (comes with Node.js)

### Recommended

- **Git** (for version control features)
- **Code editor** (VS Code, IntelliJ, etc.) for viewing changes
- **Basic terminal knowledge** (navigating directories, running commands)

### Banking IT Considerations

- **Network Access**: Ensure access to `https://api.anthropic.com`
- **Proxy Configuration**: May need corporate proxy settings
- **Firewall**: Port 443 (HTTPS) must be accessible
- **SSO/Authentication**: Check with IT for enterprise authentication requirements

---

## Installation

Claude Code can be installed in three ways. Choose the method that works best for your environment.

### Method 1: npm (Node.js Package Manager)

**Best for:** Developers who already have Node.js installed

#### Prerequisites
```bash
# Check if Node.js is installed (need 18.0+)
node --version

# Check if npm is installed
npm --version
```

#### Installation Steps

```bash
# Install globally
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

**Banking IT Note**: If your organization uses a private npm registry, you may need to configure npm:

```bash
# Set npm registry (if required)
npm config set registry https://your-internal-registry.bank.com/

# Or install with specific registry
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmjs.org/
```

#### Behind Corporate Proxy

```bash
# Set proxy for npm
npm config set proxy http://proxy.bank.com:8080
npm config set https-proxy http://proxy.bank.com:8080

# Then install
npm install -g @anthropic-ai/claude-code
```

---

### Method 2: Homebrew (macOS/Linux)

**Best for:** macOS users and Linux users with Homebrew

#### Prerequisites
```bash
# Check if Homebrew is installed
brew --version

# If not installed, visit: https://brew.sh
```

#### Installation Steps

```bash
# Install Claude Code
brew install --cask claude-code

# Verify installation
claude --version
```

**Note**: Homebrew installation includes native binaries and doesn't require Node.js.

---

### Method 3: Native Install (Platform-Specific)

**Best for:** Users who don't have Node.js or Homebrew

#### macOS (Intel/Apple Silicon)

```bash
# Download and install (auto-detects architecture)
curl -fsSL https://install.claude.ai/claude-code | sh

# Or download manually from:
# Intel: https://download.claude.ai/claude-code/macos/x64/latest
# Apple Silicon: https://download.claude.ai/claude-code/macos/arm64/latest
```

#### Linux

```bash
# Download and install
curl -fsSL https://install.claude.ai/claude-code | sh

# Or specify architecture:
# x64:
curl -fsSL https://install.claude.ai/claude-code/linux/x64/latest -o claude-code.tar.gz

# ARM64:
curl -fsSL https://install.claude.ai/claude-code/linux/arm64/latest -o claude-code.tar.gz

# Extract and install
tar -xzf claude-code.tar.gz
sudo mv claude /usr/local/bin/
```

#### Windows

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

**Banking IT Note**: WSL2 provides better isolation and is recommended for Windows users in banking environments.

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

```bash
# Set API key as environment variable
export ANTHROPIC_API_KEY="your-api-key-here"

# Or add to your shell profile (~/.bashrc, ~/.zshrc)
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.zshrc
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

**Solution:**

```bash
# Check if Claude is in PATH
which claude

# If not found, add to PATH (npm installation)
echo 'export PATH="$PATH:$(npm bin -g)"' >> ~/.zshrc
source ~/.zshrc

# For Homebrew installation
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Issue: "Node version not supported"

**Solution:**

```bash
# Check Node version
node --version

# If < 18.0, update Node.js
# Using nvm (Node Version Manager)
nvm install 18
nvm use 18

# Or download from: https://nodejs.org
```

### Issue: "Permission denied" during npm install

**Solution:**

```bash
# Option 1: Use sudo (not recommended)
sudo npm install -g @anthropic-ai/claude-code

# Option 2: Fix npm permissions (recommended)
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Then install again
npm install -g @anthropic-ai/claude-code
```

### Issue: "Cannot connect to api.anthropic.com"

**Banking IT Solutions:**

```bash
# Check network connectivity
curl -I https://api.anthropic.com

# If behind corporate proxy, set proxy
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080

# Add to shell profile for persistence
echo 'export HTTP_PROXY=http://proxy.bank.com:8080' >> ~/.zshrc
echo 'export HTTPS_PROXY=http://proxy.bank.com:8080' >> ~/.zshrc
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

**Banking IT Solution:**

```bash
# If using corporate SSL inspection
# Set custom CA certificate
export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca-bundle.crt

# Add to shell profile
echo 'export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca-bundle.crt' >> ~/.zshrc
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
- Three installation methods (npm, Homebrew, native)
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

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** None (entry-level document)
