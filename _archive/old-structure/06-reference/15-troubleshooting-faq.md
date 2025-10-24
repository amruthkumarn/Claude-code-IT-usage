# Section 15: Troubleshooting & FAQ

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [Authentication Problems](#authentication-problems)
3. [Permission Errors](#permission-errors)
4. [Performance Issues](#performance-issues)
5. [Network and Proxy](#network-and-proxy)
6. [Frequently Asked Questions](#frequently-asked-questions)

---

## Installation Issues

### Issue: "command not found: claude"

**Cause:** Claude not in PATH

**Solution (PowerShell):**
```powershell
# Check if installed
Get-Command claude

# If not found, add to PATH (pip installation)
$pipPath = python -m site --user-base
$env:Path += ";$pipPath\Scripts"
Add-Content $PROFILE "`$env:Path += `";$pipPath\Scripts`""

# Verify
claude --version
```

**Solution (WSL2):**
```bash
# Check if installed
which claude

# If not found, add to PATH (pip installation)
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
source ~/.bashrc

# Verify
claude --version
```

### Issue: "Python version not supported"

**Cause:** Python < 3.9

**Solution:**
```bash
# Check version
python --version

# Update Python
# Option 1: pyenv
pyenv install 3.9
pyenv global 3.9

# Option 2: Download from python.org
# https://python.org

# Verify
python --version  # Should be 3.9+
```

### Issue: "Permission denied" during pip install

**Cause:** Insufficient permissions

**Solution (PowerShell):**
```powershell
# Run PowerShell as Administrator, then install
pip install claude-code

# Or use user installation (recommended)
pip install --user claude-code
```

**Solution (WSL2):**
```bash
# Option 1: User installation (recommended)
pip install --user claude-code

# Option 2: Virtual environment (best practice for banking projects)
python -m venv ~/.venvs/claude-env
source ~/.venvs/claude-env/bin/activate
pip install claude-code

# Option 3: Use sudo (not recommended)
sudo pip install claude-code
```

---

## Authentication Problems

### Issue: "Authentication failed"

**Cause:** Invalid or expired credentials

**Solution:**
```bash
# Clear existing credentials
rm -rf ~/.claude/credentials.json

# Login again
claude /login

# Or set API key
export ANTHROPIC_API_KEY="your-api-key"
```

### Issue: "API key not found"

**Cause:** API key not set

**Solution:**
```bash
# Set API key
export ANTHROPIC_API_KEY="sk-ant-..."

# Add to shell profile for persistence
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc

# Verify
echo $ANTHROPIC_API_KEY
```

### Issue: "Rate limit exceeded"

**Cause:** Too many API requests

**Solution:**
```bash
# Wait a few minutes and retry

# Or use different account/API key

# Check usage at:
# https://console.anthropic.com
```

---

## Permission Errors

### Issue: "Permission denied: Cannot write to file"

**Cause:** File not in working directory scope

**Solution:**
```bash
# Start Claude in correct directory
cd /path/to/project
claude

# Or add directory to scope
claude --add-dir /path/to/other/directory
```

### Issue: "Tool 'Bash' is not allowed"

**Cause:** Bash tool blocked in settings

**Solution:**
```json
// .claude/settings.json
{
  "permissions": {
    "allow": ["Bash"],  // or
    "requireApproval": ["Bash"]  // Add Bash
  }
}
```

### Issue: "Cannot access parent directory"

**Cause:** Security restriction

**Solution:**
```bash
# Add parent directory explicitly
claude --add-dir ../parent-directory

# Or start Claude in parent directory
cd ..
claude
```

---

## Performance Issues

### Issue: Claude is slow to respond

**Causes & Solutions:**

**1. Context too large:**
```bash
# Start fresh session
Ctrl+D
claude

# Use focused queries
> Explain only the auth module
# Instead of: > Explain entire codebase
```

**2. Network latency:**
```bash
# Check network
ping api.anthropic.com

# Check proxy settings
echo $HTTP_PROXY
```

### Issue: "Context window exceeded"

**Cause:** Too much information in context

**Solution:**
```bash
# Start new session
Ctrl+D
claude

# Be more specific
> Focus on src/auth/ directory only

# Use agents for subtasks (fresh context each)
> @security-auditor review security
```

### Issue: Claude "forgets" earlier information

**Cause:** Context limit reached

**Solution:**
```bash
# Add to CLAUDE.md for permanent memory
# Instead of repeating in conversation

# Or start fresh session
# Use --continue if context needed
```

---

## Network and Proxy

### Issue: "Cannot connect to api.anthropic.com"

**Cause:** Network or firewall blocking

**Solution (PowerShell):**
```powershell
# Test connectivity
curl -I https://api.anthropic.com

# If behind corporate proxy
$env:HTTP_PROXY = "http://proxy.company.com:8080"
$env:HTTPS_PROXY = "http://proxy.company.com:8080"
$env:NO_PROXY = "localhost,127.0.0.1"

# Add to PowerShell profile
Add-Content $PROFILE 'Set-Item -Path Env:HTTP_PROXY -Value "http://proxy.company.com:8080"'
Add-Content $PROFILE 'Set-Item -Path Env:HTTPS_PROXY -Value "http://proxy.company.com:8080"'
```

**Solution (WSL2):**
```bash
# Test connectivity
curl -I https://api.anthropic.com

# If behind corporate proxy
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
export NO_PROXY="localhost,127.0.0.1"

# Add to shell profile
echo 'export HTTP_PROXY="http://proxy.company.com:8080"' >> ~/.bashrc
echo 'export HTTPS_PROXY="http://proxy.company.com:8080"' >> ~/.bashrc
```

### Issue: "SSL certificate error"

**Cause:** Corporate SSL inspection (common in banking environments)

**Solution:**
```bash
# Set custom CA certificate for Python/pip
export REQUESTS_CA_BUNDLE="/path/to/company-ca.crt"
export SSL_CERT_FILE="/path/to/company-ca.crt"

# For pip specifically
export PIP_CERT="/path/to/company-ca.crt"

# Add to shell profile
echo 'export REQUESTS_CA_BUNDLE="/path/to/company-ca.crt"' >> ~/.zshrc
echo 'export SSL_CERT_FILE="/path/to/company-ca.crt"' >> ~/.zshrc
echo 'export PIP_CERT="/path/to/company-ca.crt"' >> ~/.zshrc

# Verify
python -c "import os; print(os.environ.get('REQUESTS_CA_BUNDLE'))"
```

### Issue: "Timeout connecting to API"

**Cause:** Slow network or proxy

**Solution:**
```bash
# Increase timeout
claude --timeout 300000  # 5 minutes (default: 2 minutes)

# Check network speed
curl -w "@-" -o /dev/null -s https://api.anthropic.com << 'EOF'
     time_total:  %{time_total}\n
EOF
```

### Issue: "pip install fails behind corporate proxy"

**Cause:** Banking firewall/proxy blocking pip repository

**Solution:**
```bash
# Configure pip to use corporate proxy
pip install --proxy http://proxy.company.com:8080 package-name

# Or set environment variables
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"
pip install package-name

# For trusted host (if SSL inspection causes issues)
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org package-name

# Configure pip permanently
pip config set global.proxy http://proxy.company.com:8080
pip config set global.trusted-host "pypi.org files.pythonhosted.org"
```

### Issue: "Cannot import PySpark modules"

**Cause:** PySpark not installed or environment issues

**Solution:**
```bash
# Install PySpark
pip install pyspark

# Verify installation
python -c "import pyspark; print(pyspark.__version__)"

# Set environment variables for Databricks
export DATABRICKS_HOST="https://your-workspace.cloud.databricks.com"
export DATABRICKS_TOKEN="your-token"

# Test connection
python -c "from databricks import sql; print('Databricks SDK ready')"
```

### Issue: "PySpark job fails with Java errors"

**Cause:** Java not installed or incorrect version

**Solution:**
```bash
# Check Java version
java -version  # Should be Java 8 or 11

# Set JAVA_HOME
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk"
echo 'export JAVA_HOME="/usr/lib/jvm/java-11-openjdk"' >> ~/.bashrc

# Set SPARK_HOME (if using local Spark)
export SPARK_HOME="/opt/spark"
export PATH="$SPARK_HOME/bin:$PATH"

# Verify
pyspark --version
```

---

## Frequently Asked Questions

### General Questions

**Q: Is my code sent to Anthropic?**

A: Yes, Claude Code sends your code to Anthropic's API for processing. Use on-premises deployment for sensitive code.

**Q: Can I use Claude Code offline?**

A: No, Claude Code requires internet connection to Anthropic's API.

**Q: How much does Claude Code cost?**

A: Pricing is based on API usage (tokens). See: https://www.anthropic.com/pricing

**Q: What's the difference between models?**

A:
- Sonnet: Balanced (default)
- Opus: Most capable, slower, expensive
- Haiku: Fastest, cheapest, less capable

**Q: Can I use Claude Code in CI/CD?**

A: Yes, using `claude -p` for non-interactive mode:
```bash
claude -p "Run security scan" > report.txt
```

### Configuration Questions

**Q: Where are settings stored?**

A:
- User: `~/.claude/settings.json`
- Project: `.claude/settings.json`
- Local: `.claude/settings.local.json`

**Q: How do I share configuration with team?**

A: Commit `.claude/settings.json` to git. Add `.claude/settings.local.json` to `.gitignore`.

**Q: Can I have different settings per branch?**

A: Yes, git will switch `.claude/settings.json` when you change branches.

### Security Questions

**Q: Is Claude Code secure for banking applications?**

A: Claude Code has security features (read-only default, approval workflows), but:
- Review all changes before approval
- Use permission restrictions
- Enable audit logging
- Never auto-approve in production
- Follow your organization's security policies

**Q: How do I prevent secrets from being committed?**

A: Use pre-commit hooks:
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

**Q: Can I restrict Claude to read-only?**

A: Yes:
```bash
claude --permission-mode plan
```

Or in settings:
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "Bash"]
  }
}
```

### Performance Questions

**Q: Why is Claude slow?**

A: Common causes:
- Large context window
- Network latency
- Solution: Keep context focused, start fresh sessions

**Q: How can I speed up Claude?**

A:
- Start fresh sessions regularly (Ctrl+D, then `claude`)
- Be specific in queries
- Use agents for subtasks
- Keep working directory scope narrow

**Q: What's the maximum file size Claude can read?**

A: No hard limit, but large files use more context tokens. Files > 10,000 lines may hit context limits.

### Workflow Questions

**Q: Can Claude create git commits?**

A: Yes:
```
> Create a commit for these changes
```

Claude will draft message and execute git commands (with approval).

**Q: Can Claude push to remote?**

A: Yes, but requires approval:
```
> Create a pull request
```

**Q: Should I let Claude run tests?**

A: Yes, but review results:
```
> Run the test suite

[Claude executes: pytest tests/]
[Review test output before proceeding]
```

**Q: Can I continue a session later?**

A: Yes:
```bash
claude --continue
```

### Banking & PySpark Questions

**Q: How do I troubleshoot PySpark dependency issues?**

A: Check dependencies systematically:
```bash
# List installed packages
pip list | grep -i spark

# Check for conflicts
pip check

# Reinstall with specific versions
pip install pyspark==3.4.0 delta-spark==2.4.0

# Use requirements.txt for reproducibility
pip install -r requirements.txt
```

**Q: How do I debug PySpark jobs in Databricks?**

A: Use logging and Databricks utilities:
```python
# Add logging to PySpark code
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info(f"Processing {df.count()} records")

# Use display() for debugging in Databricks notebooks
display(df.limit(10))

# Check Spark UI for detailed execution plans
# Available in Databricks cluster UI
```

**Q: How do I handle SSL certificate errors with banking APIs?**

A: Configure Python/requests to use corporate certificates:
```python
# In your Python code
import requests
import os

# Set certificate path
cert_path = "/path/to/company-ca.crt"
os.environ['REQUESTS_CA_BUNDLE'] = cert_path

# Make requests with certificate
response = requests.get(
    "https://internal-api.bank.com",
    verify=cert_path
)

# Or disable verification (not recommended for production)
response = requests.get(url, verify=False)
```

**Q: How do I troubleshoot poetry/pip installation conflicts?**

A: Use virtual environments and dependency resolution:
```bash
# Create fresh virtual environment
python -m venv venv
source venv/bin/activate

# Install with pip
pip install -r requirements.txt

# Or use poetry for better dependency resolution
poetry install

# Check for conflicts
pip check

# Export poetry dependencies to requirements.txt
poetry export -f requirements.txt --output requirements.txt
```

### Troubleshooting Questions

**Q: Claude gives incorrect answers**

A: Try:
- Be more specific in your question
- Start fresh session (clean context)
- Use Opus model for complex questions
- Provide more context in CLAUDE.md

**Q: Changes aren't being applied**

A: Check:
- Did you approve the changes?
- Are permissions configured correctly?
- Is file writable?
- Check `.claude/settings.json`

**Q: Custom commands not working**

A: Verify:
- File is in `.claude/commands/`
- File ends in `.md`
- Frontmatter is correct
- Run `/help` to see if command listed

**Q: pytest tests failing with import errors**

A: Fix Python path and imports:
```bash
# Ensure PYTHONPATH includes src directory
export PYTHONPATH="${PYTHONPATH}:${PWD}/src"

# Or install package in editable mode
pip install -e .

# Run tests with verbose output
pytest -v tests/

# Show stdout for debugging
pytest -v -s tests/

# Run specific test file
pytest tests/test_transactions.py
```

**Q: Ruff linting errors in banking project**

A: Configure Ruff for banking standards:
```bash
# Run Ruff linter
ruff check src/

# Auto-fix safe issues
ruff check --fix src/

# Check specific file
ruff check src/banking/transactions.py

# Configure in pyproject.toml for consistent team standards
# [tool.ruff]
# line-length = 100
# select = ["E", "F", "I", "N"]
```

**Q: Virtual environment not activating**

A: Check activation commands:
```bash
# Create venv
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Verify activation
which python  # Should show venv path
pip list  # Should show venv packages
```

**Q: requirements.txt vs pyproject.toml conflicts**

A: Choose one dependency management approach:
```bash
# Option 1: Use requirements.txt (simple)
pip install -r requirements.txt
pip freeze > requirements.txt

# Option 2: Use pyproject.toml with poetry (recommended for banking)
poetry install
poetry add pyspark
poetry export -f requirements.txt --output requirements.txt

# Option 3: Use pyproject.toml with pip
pip install -e .

# Check for conflicts
pip check
```

---

## Getting Help

### Official Resources

- **Documentation**: https://docs.claude.com/en/docs/claude-code/overview
- **GitHub Issues**: https://github.com/anthropics/claude-code/issues
- **Support**: https://support.anthropic.com

### Enable Debug Mode

```bash
# Enable verbose output
claude --verbose

# Enable debug logging
export CLAUDE_DEBUG=1
claude

# Check logs
tail -f ~/.claude/logs/*.log
```

### Python Debugging

```bash
# Run Python with debugger
python -m pdb script.py

# Or use ipdb (enhanced debugger)
pip install ipdb
python -m ipdb script.py

# Debug PySpark jobs with logging
export SPARK_LOG_LEVEL=DEBUG
pyspark

# Add breakpoint in code (Python 3.7+)
# In your .py file:
breakpoint()  # Execution will pause here

# Run pytest with debugging on failure
pytest --pdb tests/

# Debug specific test
pytest --pdb tests/test_transactions.py::test_validate_amount
```

### Reporting Issues

Include:
1. Claude Code version: `claude --version`
2. Operating system
3. Python version: `python --version`
4. Error message (full traceback)
5. Steps to reproduce
6. `.claude/settings.json` (remove secrets!)

---

## Summary

This section covered:

### Installation
- PATH issues
- Python version requirements
- pip/poetry permission problems
- Virtual environment setup

### Authentication
- Login failures
- API key configuration
- Rate limiting

### Permissions
- File access restrictions
- Tool blocking
- Directory scope

### Performance
- Model selection
- Context management
- Network optimization

### Banking-Specific Issues
- Corporate proxy configuration
- SSL certificate errors (banking SSL inspection)
- PySpark dependency management
- Databricks connection issues
- pip installation behind firewalls

### Common Questions
- Security concerns
- Configuration management
- Workflow best practices
- pytest and Ruff troubleshooting
- Python dependency management

---


**Additional Resources:**
- **Official Documentation**: https://docs.claude.com/en/docs/claude-code/overview
- **Troubleshooting Guide**: https://docs.claude.com/en/docs/claude-code/troubleshooting
- **Community Forum**: https://community.anthropic.com
