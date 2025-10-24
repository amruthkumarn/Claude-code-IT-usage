# Phase 1.2: Installation

**Learning Objectives:**
- Install Claude Code for banking IT environment
- Configure for corporate proxies and certificates
- Authenticate successfully
- Verify installation is working
- Troubleshoot common issues

**Time Commitment:** 1-2 hours

**Prerequisites:**
- ✅ Completed Phase 1.1 (Introduction)
- ✅ Python 3.9+ installed
- ✅ Terminal access (PowerShell or WSL2)
- ✅ Network access to api.anthropic.com

---

## Table of Contents
1. [Pre-Installation Checklist](#pre-installation-checklist)
2. [Installation Method 1: pip](#installation-method-1-pip-recommended)
3. [Installation Method 2: Native Windows](#installation-method-2-native-windows)
4. [Banking IT Specific Configuration](#banking-it-specific-configuration)
5. [Authentication & Login](#authentication--login)
6. [Verification](#verification)
7. [Post-Installation Setup](#post-installation-setup)
8. [Troubleshooting](#troubleshooting)
9. [Success Criteria](#success-criteria)

---

## Pre-Installation Checklist

Before installing Claude Code, verify you have the following:

### Required

**1. Python 3.9 or later**
```bash
# Check Python version
python --version
# Should show: Python 3.9.x or higher

# If not installed, download from: https://www.python.org/downloads/
```

**2. pip (Python package manager)**
```bash
# Check pip version
pip --version
# Should show: pip 21.x or higher
```

**3. Terminal Access**
- **Windows**: PowerShell 5.1+ or WSL2 (Ubuntu recommended)
- **Access Level**: Standard user (admin not required for pip --user installation)

**4. Claude Account**
- Claude.ai account OR
- Claude Console account (for enterprise)
- API access enabled

### Banking IT Specific

**5. Network Access**
```bash
# Test connectivity to Anthropic API
curl -I https://api.anthropic.com

# Expected: HTTP/2 200 or 301/302
# If fails: Check with IT for proxy/firewall configuration
```

**6. Corporate Proxy Information** (if applicable)
- Proxy URL: `http://proxy.bank.com:8080`
- Proxy authentication (if required)
- No-proxy exceptions

**7. SSL Certificates** (if using SSL inspection)
- Corporate CA bundle location
- Certificate trust configuration

**8. Disk Space**
```bash
# Claude Code requires ~100MB
# Check available space
df -h .   # Linux/WSL2
# or
Get-PSDrive C   # PowerShell
```

### Recommended

- **Git** installed (for version control features)
- **Virtual environment** tool (venv, virtualenv, or conda)
- **Code editor** (VS Code recommended)

---

## Installation Method 1: pip (Recommended)

**Best for:** Most users, especially those with Python already installed

### Step 1: Verify Prerequisites

```bash
# Verify Python 3.9+
python --version

# Verify pip
pip --version
```

### Step 2: Install Claude Code

**Option A: User Installation (Recommended for Banking IT)**
```bash
# Install for current user only (no admin required)
pip install --user anthropic-claude-code

# Verify installation
claude --version
```

**Option B: Global Installation** (requires admin)
```bash
# PowerShell (Run as Administrator)
pip install anthropic-claude-code

# Or WSL2 with sudo
sudo pip install anthropic-claude-code
```

**Option C: Virtual Environment** (Best Practice)
```bash
# Create virtual environment for Claude Code
python -m venv ~/.venv/claude-code

# Activate (WSL2/Linux/Mac)
source ~/.venv/claude-code/bin/activate

# Activate (PowerShell)
~\.venv\claude-code\Scripts\Activate.ps1

# Install
pip install anthropic-claude-code

# Verify
claude --version
```

### Step 3: Add to PATH (if needed)

If `claude` command not found:

**PowerShell:**
```powershell
# Get Python user base directory
$pythonPath = python -c "import site; print(site.USER_BASE)"

# Add Scripts directory to PATH
$env:Path += ";$pythonPath\Scripts"

# Make permanent (add to PowerShell profile)
Add-Content $PROFILE "`$env:Path += `";$pythonPath\Scripts`""
```

**WSL2/Linux:**
```bash
# Add to PATH
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
source ~/.bashrc

# Verify
which claude
```

### Step 4: Verify Installation

```bash
# Check version
claude --version
# Expected: claude-code version 1.x.x

# Check help
claude --help
```

---

## Installation Method 2: Native Windows

### Option A: PowerShell Script

```powershell
# Run PowerShell as Administrator
# Right-click PowerShell → Run as Administrator

# Install via install script
irm https://install.claude.ai/claude-code/windows | iex

# Verify
claude --version
```

### Option B: WSL2 (Recommended for Banking IT)

**Why WSL2?**
- Better isolation from Windows
- Linux environment familiar to most data engineers
- Easier PySpark development
- Better compatibility with banking tools

**Installation:**

**Step 1: Enable WSL2**
```powershell
# In PowerShell as Administrator
wsl --install

# Restart computer if prompted
```

**Step 2: Install Ubuntu**
```powershell
# Install Ubuntu from Microsoft Store
# Or via command line:
wsl --install -d Ubuntu-22.04
```

**Step 3: Install Claude Code in WSL2**
```bash
# Inside WSL2 Ubuntu terminal
curl -fsSL https://install.claude.ai/claude-code | sh

# Verify
claude --version
```

---

## Banking IT Specific Configuration

### 1. Corporate Proxy Setup

**PowerShell (Windows):**
```powershell
# Set proxy for current session
$env:HTTP_PROXY = "http://proxy.bank.com:8080"
$env:HTTPS_PROXY = "http://proxy.bank.com:8080"

# Set proxy with authentication
$env:HTTP_PROXY = "http://username:password@proxy.bank.com:8080"
$env:HTTPS_PROXY = "http://username:password@proxy.bank.com:8080"

# Make permanent (add to PowerShell profile)
Add-Content $PROFILE @'
$env:HTTP_PROXY = "http://proxy.bank.com:8080"
$env:HTTPS_PROXY = "http://proxy.bank.com:8080"
'@

# Reload profile
. $PROFILE
```

**WSL2/Linux:**
```bash
# Set proxy for current session
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080

# With authentication
export HTTP_PROXY=http://username:password@proxy.bank.com:8080
export HTTPS_PROXY=http://username:password@proxy.bank.com:8080

# Make permanent (add to .bashrc)
cat >> ~/.bashrc <<'EOF'
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080
export NO_PROXY=localhost,127.0.0.1,.bank.internal
EOF

# Reload
source ~/.bashrc
```

**Configure pip for proxy:**
```bash
# Set proxy in pip configuration
pip config set global.proxy http://proxy.bank.com:8080

# Verify
pip config list
```

### 2. SSL Certificate Configuration

If your bank uses SSL inspection:

**PowerShell:**
```powershell
# Set custom CA bundle
$env:REQUESTS_CA_BUNDLE = "C:\certificates\corporate-ca-bundle.crt"
$env:SSL_CERT_FILE = "C:\certificates\corporate-ca-bundle.crt"
$env:PIP_CERT = "C:\certificates\corporate-ca-bundle.crt"

# Make permanent
Add-Content $PROFILE @'
$env:REQUESTS_CA_BUNDLE = "C:\certificates\corporate-ca-bundle.crt"
$env:SSL_CERT_FILE = "C:\certificates\corporate-ca-bundle.crt"
$env:PIP_CERT = "C:\certificates\corporate-ca-bundle.crt"
'@
```

**WSL2/Linux:**
```bash
# Set CA bundle location
export REQUESTS_CA_BUNDLE=/etc/ssl/certs/corporate-ca-bundle.crt
export SSL_CERT_FILE=/etc/ssl/certs/corporate-ca-bundle.crt
export PIP_CERT=/etc/ssl/certs/corporate-ca-bundle.crt

# Add to .bashrc for persistence
cat >> ~/.bashrc <<'EOF'
export REQUESTS_CA_BUNDLE=/etc/ssl/certs/corporate-ca-bundle.crt
export SSL_CERT_FILE=/etc/ssl/certs/corporate-ca-bundle.crt
export PIP_CERT=/etc/ssl/certs/corporate-ca-bundle.crt
EOF
```

**Install certificates (if needed):**
```bash
# Copy corporate CA bundle
sudo cp /path/to/corporate-ca-bundle.crt /usr/local/share/ca-certificates/

# Update certificate store
sudo update-ca-certificates
```

### 3. Private PyPI Registry

If your organization uses a private PyPI mirror:

```bash
# Configure pip to use internal PyPI
pip config set global.index-url https://pypi.bank.internal/simple/

# Trust internal host
pip config set global.trusted-host pypi.bank.internal

# Or specify during installation
pip install anthropic-claude-code --index-url https://pypi.bank.internal/simple/ --trusted-host pypi.bank.internal

# Verify configuration
pip config list
```

---

## Authentication & Login

After installation, authenticate Claude Code with your Claude account.

### First-Time Login

```bash
# Start login process
claude /login
```

**What happens:**
1. Browser window opens automatically
2. Redirects to `https://claude.ai/login`
3. You enter your credentials (or use SSO)
4. Authentication token generated
5. Token stored securely in system keychain
6. Terminal shows: "Successfully logged in as: your-email@bank.com"

### Login Flow Diagram

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
│  Banking IT: SSO Login (SAML/OIDC)          │
│  - Enter corporate credentials               │
│  - Multi-factor authentication (if required) │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  Authorization granted                        │
│  Token stored in:                             │
│  - Windows: Credential Manager                │
│  - WSL2/Linux: ~/.claude/credentials.json    │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  Terminal shows: "Successfully logged in"     │
│  Ready to use: claude                         │
└──────────────────────────────────────────────┘
```

### Alternative: API Key Authentication

For enterprise users or CI/CD environments:

**Get API Key:**
1. Go to https://console.anthropic.com (enterprise) or https://claude.ai/settings
2. Navigate to API Keys
3. Create new API key
4. Copy the key (starts with `sk-ant-...`)

**Set API Key:**

**PowerShell:**
```powershell
# Current session only
$env:ANTHROPIC_API_KEY = "sk-ant-your-api-key-here"

# Permanent (add to profile)
Add-Content $PROFILE 'Set-Item -Path Env:ANTHROPIC_API_KEY -Value "sk-ant-your-api-key-here"'

# Reload profile
. $PROFILE

# Verify
echo $env:ANTHROPIC_API_KEY
```

**WSL2/Linux:**
```bash
# Current session only
export ANTHROPIC_API_KEY="sk-ant-your-api-key-here"

# Permanent (add to .bashrc)
echo 'export ANTHROPIC_API_KEY="sk-ant-your-api-key-here"' >> ~/.bashrc

# Reload
source ~/.bashrc

# Verify
echo $ANTHROPIC_API_KEY
```

### Enterprise SSO Login

If your organization uses Single Sign-On:

```bash
# Enterprise login
claude /login --enterprise

# Or specify enterprise URL
claude /login --url https://claude.bank-enterprise.com
```

Contact your IT administrator for:
- Enterprise Claude Console URL
- SSO configuration (SAML, OIDC)
- Group permissions and access policies

---

## Verification

### Step-by-Step Verification

**1. Check Version**
```bash
claude --version

# Expected output:
# claude-code version 1.x.x
```

**2. Check Authentication**
```bash
claude /login

# If logged in:
# Already logged in as: your-email@bank.com

# If not logged in:
# [Browser opens for login]
```

**3. Test Basic Command**
```bash
# Start Claude Code in current directory
claude --help

# Should show command options without errors
```

**4. Test First Session**
```bash
# Navigate to a Python project
cd ~/projects/test-project

# Start Claude Code
claude

# Expected:
# Welcome to Claude Code!
# You're in: /home/yourname/projects/test-project
#
# Type /help for available commands
# >

# Exit with Ctrl+D
```

**5. Test Simple Query**
```bash
# Start Claude Code
claude

# Ask a question
> What is 2 + 2?

# Claude should respond: "4" or "2 + 2 = 4"

# Exit
Ctrl+D
```

### Verification Checklist

- [ ] `claude --version` shows version number
- [ ] `claude /login` confirms login status
- [ ] `claude --help` shows command options
- [ ] Can start Claude Code in a directory
- [ ] Can ask a simple question and get response
- [ ] Can exit cleanly with Ctrl+D

**If all checks pass: ✅ Installation successful!**

---

## Post-Installation Setup

### 1. Create Test Project

```bash
# Create a test PySpark project
mkdir -p ~/projects/claude-test
cd ~/projects/claude-test

# Create a simple Python file
cat > hello.py <<'EOF'
def greet(name: str) -> str:
    """Greet someone by name."""
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("Data Engineer"))
EOF

# Test Claude Code
claude

# In Claude Code, ask:
> Explain what this code does

# Should read hello.py and explain it
```

### 2. Configure for PySpark Development

```bash
# Set PySpark environment variables
# PowerShell:
$env:PYSPARK_PYTHON = "python"
$env:SPARK_HOME = "C:\path\to\spark"

# WSL2/Linux:
export PYSPARK_PYTHON=python3
export SPARK_HOME=/opt/spark

# Add to profile for persistence
```

### 3. Test with PySpark Code

```bash
cd ~/projects/claude-test

# Create PySpark test file
cat > pyspark_test.py <<'EOF'
from pyspark.sql import SparkSession

def create_spark_session() -> SparkSession:
    """Create SparkSession for local testing."""
    return SparkSession.builder \
        .master("local[2]") \
        .appName("ClaudeTest") \
        .getOrCreate()

if __name__ == "__main__":
    spark = create_spark_session()
    print(f"Spark version: {spark.version}")
    spark.stop()
EOF

# Test with Claude Code
claude

> Add a function to read CSV files with this SparkSession

# Claude should generate PySpark code
```

### 4. Set Up Project Template

```bash
# Create .claude directory for project configuration
mkdir -p ~/projects/claude-test/.claude

# Create basic settings file
cat > ~/projects/claude-test/.claude/settings.json <<'EOF'
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "requireApproval": ["Edit", "Write", "Bash"]
  },
  "defaultModel": "sonnet"
}
EOF

# Test
cd ~/projects/claude-test
claude

# Claude will now use these settings
```

---

## Troubleshooting

### Issue 1: "command not found: claude"

**Diagnosis:**
```bash
# Check if installed
pip show anthropic-claude-code

# Check PATH
echo $PATH  # Linux/WSL2
echo $env:Path  # PowerShell
```

**Solution (PowerShell):**
```powershell
# Find Python user base
python -c "import site; print(site.USER_BASE)"

# Add to PATH
$pythonPath = python -c "import site; print(site.USER_BASE)"
$env:Path += ";$pythonPath\Scripts"

# Make permanent
Add-Content $PROFILE "`$env:Path += `";$pythonPath\Scripts`""
```

**Solution (WSL2):**
```bash
# Add ~/.local/bin to PATH
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
source ~/.bashrc

# Verify
which claude
```

---

### Issue 2: "Python version not supported"

**Diagnosis:**
```bash
python --version
# If < 3.9: Need to upgrade
```

**Solution (Using pyenv):**
```bash
# Install pyenv
curl https://pyenv.run | bash

# Install Python 3.11
pyenv install 3.11.6

# Set as global default
pyenv global 3.11.6

# Verify
python --version
```

**Solution (Direct Download):**
- Download Python 3.11 from https://www.python.org/downloads/
- Install (ensure "Add to PATH" is checked)
- Restart terminal
- Verify: `python --version`

---

### Issue 3: "Permission denied" during installation

**Solution (User installation):**
```bash
# Install for current user only
pip install --user anthropic-claude-code

# No admin required
```

**Solution (Virtual environment):**
```bash
# Create virtual environment
python -m venv ~/.venv/claude

# Activate
source ~/.venv/claude/bin/activate  # Linux/WSL2
~\.venv\claude\Scripts\Activate.ps1  # PowerShell

# Install
pip install anthropic-claude-code
```

---

### Issue 4: "Cannot connect to api.anthropic.com"

**Diagnosis:**
```bash
# Test connectivity
curl -I https://api.anthropic.com

# If fails: Proxy or firewall issue
```

**Solution (Corporate Proxy):**

**PowerShell:**
```powershell
# Set proxy
$env:HTTP_PROXY = "http://proxy.bank.com:8080"
$env:HTTPS_PROXY = "http://proxy.bank.com:8080"

# Test
curl -I https://api.anthropic.com

# If works, make permanent
Add-Content $PROFILE @'
$env:HTTP_PROXY = "http://proxy.bank.com:8080"
$env:HTTPS_PROXY = "http://proxy.bank.com:8080"
'@
```

**WSL2/Linux:**
```bash
# Set proxy
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080

# Test
curl -I https://api.anthropic.com

# If works, make permanent
cat >> ~/.bashrc <<'EOF'
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080
EOF
```

---

### Issue 5: SSL Certificate Errors

**Error:**
```
SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed
```

**Solution (Corporate CA Bundle):**

**PowerShell:**
```powershell
# Set CA bundle
$env:REQUESTS_CA_BUNDLE = "C:\path\to\corporate-ca-bundle.crt"
$env:SSL_CERT_FILE = "C:\path\to\corporate-ca-bundle.crt"

# Test
claude --version

# Make permanent
Add-Content $PROFILE @'
$env:REQUESTS_CA_BUNDLE = "C:\path\to\corporate-ca-bundle.crt"
$env:SSL_CERT_FILE = "C:\path\to\corporate-ca-bundle.crt"
'@
```

**WSL2/Linux:**
```bash
# Copy CA bundle to system location
sudo cp /path/to/corporate-ca-bundle.crt /usr/local/share/ca-certificates/corporate-ca.crt

# Update certificates
sudo update-ca-certificates

# Set environment variables
export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt

# Make permanent
echo 'export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt' >> ~/.bashrc
echo 'export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt' >> ~/.bashrc
```

---

### Issue 6: "Authentication failed"

**Solution:**
```bash
# Clear existing credentials
rm ~/.claude/credentials.json  # Linux/WSL2
# Or delete from Windows Credential Manager

# Login again
claude /login

# Follow browser authentication flow
```

---

### Issue 7: Installation behind firewall

**Requirements:**
- Access to `api.anthropic.com` (HTTPS/443)
- Access to `claude.ai` (for login)
- Access to `pypi.org` (for pip install)

**Request IT to whitelist:**
```
api.anthropic.com:443
claude.ai:443
console.anthropic.com:443
pypi.org:443
files.pythonhosted.org:443
```

---

## Success Criteria

You're ready for Phase 1.3 when you can:

- ✅ Run `claude --version` successfully
- ✅ Log in with `claude /login`
- ✅ Start Claude Code in a project directory
- ✅ Ask a question and get a response
- ✅ Exit cleanly
- ✅ Proxy/SSL configured (if needed for banking IT)
- ✅ Environment variables set for PySpark

**Installation Time:** 30-60 minutes (more if troubleshooting needed)

---

## Quick Reference

### Essential Installation Commands

```bash
# Install (recommended method)
pip install --user anthropic-claude-code

# Verify
claude --version

# Login
claude /login

# Test
claude --help

# First session
claude
```

### Banking IT Environment Variables

```bash
# Proxy
export HTTP_PROXY=http://proxy.bank.com:8080
export HTTPS_PROXY=http://proxy.bank.com:8080

# SSL Certificates
export REQUESTS_CA_BUNDLE=/path/to/ca-bundle.crt
export SSL_CERT_FILE=/path/to/ca-bundle.crt

# PySpark
export PYSPARK_PYTHON=python3
export SPARK_HOME=/opt/spark

# API Key (alternative auth)
export ANTHROPIC_API_KEY="sk-ant-your-key"
```

---

## Next Steps

**Installation complete?**

👉 **[Continue to Phase 1.3: Core Concepts & Features](./03-core-concepts-features.md)**

Learn about:
- How Claude Code works
- Security & permissions
- Memory & context
- All Claude Code features

**Need help?**
- Check [Troubleshooting section](#troubleshooting) above
- Review [FAQ](../reference/troubleshooting-faq.md)
- Ask your team lead or IT support

---

## Summary

In this section, you:
- ✅ Verified prerequisites (Python 3.9+, pip, network access)
- ✅ Installed Claude Code (pip method recommended)
- ✅ Configured for banking IT (proxy, SSL certificates)
- ✅ Authenticated successfully
- ✅ Verified installation is working
- ✅ Set up PySpark development environment
- ✅ Learned troubleshooting for common issues

**Time Spent:** ~1-2 hours (including troubleshooting)

**Next:** Understand how Claude Code works and all its features → [Phase 1.3: Core Concepts](./03-core-concepts-features.md)
