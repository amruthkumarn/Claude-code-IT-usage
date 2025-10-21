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

# If not found, add to PATH (npm installation)
$npmPath = npm config get prefix
$env:Path += ";$npmPath"
Add-Content $PROFILE "`$env:Path += `";$(npm config get prefix)`""

# Verify
claude --version
```

**Solution (WSL2):**
```bash
# Check if installed
which claude

# If not found, add to PATH (npm installation)
echo 'export PATH="$PATH:$(npm bin -g)"' >> ~/.bashrc
source ~/.bashrc

# Verify
claude --version
```

### Issue: "Node version not supported"

**Cause:** Node.js < 18.0

**Solution:**
```bash
# Check version
node --version

# Update Node.js
# Option 1: nvm
nvm install 18
nvm use 18

# Option 2: Download from nodejs.org
# https://nodejs.org

# Verify
node --version  # Should be 18.0+
```

### Issue: "Permission denied" during npm install

**Cause:** Insufficient permissions

**Solution (PowerShell):**
```powershell
# Run PowerShell as Administrator, then install
npm install -g @anthropic-ai/claude-code
```

**Solution (WSL2):**
```bash
# Option 1: Use sudo
sudo npm install -g @anthropic-ai/claude-code

# Option 2: Fix npm permissions (recommended)
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Then install
npm install -g @anthropic-ai/claude-code

# Option 2: Use sudo (not recommended)
sudo npm install -g @anthropic-ai/claude-code
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

**Cause:** Corporate SSL inspection

**Solution:**
```bash
# Set custom CA certificate
export NODE_EXTRA_CA_CERTS="/path/to/company-ca.crt"

# Add to shell profile
echo 'export NODE_EXTRA_CA_CERTS="/path/to/ca.crt"' >> ~/.zshrc

# Verify
node -e "console.log(process.env.NODE_EXTRA_CA_CERTS)"
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

[Claude executes: npm test]
[Review test output before proceeding]
```

**Q: Can I continue a session later?**

A: Yes:
```bash
claude --continue
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

### Reporting Issues

Include:
1. Claude Code version: `claude --version`
2. Operating system
3. Node.js version: `node --version`
4. Error message (full output)
5. Steps to reproduce
6. `.claude/settings.json` (remove secrets!)

---

## Summary

This section covered:

### Installation
- PATH issues
- Node.js version requirements
- Permission problems

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

### Common Questions
- Security concerns
- Configuration management
- Workflow best practices

---

**Document Version:** 1.0
**Last Updated:** 2025-10-19
**Target Audience:** Banking IT - Data Chapter
**Prerequisites:** Sections 1-14

**Additional Resources:**
- **Official Documentation**: https://docs.claude.com/en/docs/claude-code/overview
- **Troubleshooting Guide**: https://docs.claude.com/en/docs/claude-code/troubleshooting
- **Community Forum**: https://community.anthropic.com
