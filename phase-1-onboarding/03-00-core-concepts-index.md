# Phase 1.3: Core Concepts & Features - Index

**Learning Objectives:**
- Understand how Claude Code works (architecture and workflow)
- Master the security and permissions model
- Learn memory and context management
- Understand all CLI commands and features
- Configure projects with settings, slash commands, and agents
- Set up hooks for automation and compliance
- Integrate with external systems via MCP
- Understand git workflows (manual operations)
- Apply banking IT standards and best practices
- Use the templates library effectively

**Time Commitment:** 6-8 hours (comprehensive overview of all features)

**Prerequisites:**
- ✅ Completed Phase 1.1 (Introduction)
- ✅ Completed Phase 1.2 (Installation)
- ✅ Claude Code installed and authenticated

---

## Table of Contents

**Part 1: Foundation**
1. **[How Claude Code Works](./03-01-how-claude-works.md)**
   - Architecture overview
   - Request-to-response workflow
   - Key principles (conversational, tool-based, approval-based)
   - Interactive REPL vs Plan Mode

2. **[Security & Permissions Model](./03-02-security-permissions.md)**
   - Banking IT security requirements
   - Permission system (allow, requireApproval, deny)
   - Available tools and risk levels
   - Secrets management
   - Audit logging for compliance

3. **[Memory & Context Management](./03-03-memory-context.md)**
   - Types of memory (session, project, user)
   - Memory hierarchy and precedence
   - CLAUDE.md project memory
   - Banking IT CLAUDE.md template
   - Context management tips

4. **[CLI Reference & Commands](./03-04-cli-reference.md)**
   - Command-line syntax
   - Essential commands (start, print mode, version, help)
   - Permission modes (plan, standard, custom)
   - Environment variables
   - Keyboard shortcuts

**Part 2: Configuration**

5. **[Project Configuration](./03-05-project-configuration.md)**
   - The .claude directory structure
   - Settings hierarchy and precedence
   - settings.json (team-shared configuration)
   - settings.local.json (personal preferences)
   - Permission profiles for banking IT
   - Output styles customization

6. **[Slash Commands](./03-06-slash-commands.md)**
   - What are slash commands?
   - Built-in commands
   - Creating custom commands
   - Command arguments and patterns
   - Banking IT command examples (compliance check, security review, pipeline docs)

7. **[Agents & Sub-agents](./03-07-agents-subagents.md)**
   - Understanding agents and specialization
   - When to use agents
   - Defining custom agents
   - Agent configuration and tool restrictions
   - Banking data engineering agent examples (compliance checker, SQL auditor, performance optimizer)

8. **[Hooks & Automation](./03-08-hooks-automation.md)**
   - Understanding hooks and lifecycle
   - Hook types (PreToolUse, PostToolUse, UserPromptSubmit)
   - Configuring hooks
   - Banking automation examples (audit logging, secrets detection, PCI-DSS compliance)

**Part 3: Integration**

9. **[MCP Integration](./03-09-mcp-integration.md)**
   - What is Model Context Protocol (MCP)?
   - MCP server types (HTTP, SSE, stdio)
   - Installing MCP servers
   - Banking IT integrations (databases, Jira, monitoring, internal APIs)

10. **[Git Integration](./03-10-git-integration.md)**
    - Banking IT git policy (manual operations only)
    - How Claude assists with git workflows
    - Commit message generation
    - Pull request creation
    - Branch management
    - Best practices

**Part 4: Standards**

11. **[Standards & Best Practices](./03-11-standards-best-practices.md)**
    - PySpark coding standards
    - Code quality guidelines
    - Security best practices
    - Performance optimization
    - Testing standards
    - Documentation requirements

12. **[Templates Library](./03-12-templates-library.md)**
    - Available templates overview
    - BRD templates
    - Data mapping templates
    - PySpark code templates
    - Testing templates
    - CI/CD pipeline templates

---

## Learning Path

### Recommended Order

**For New Users:**
1. Start with Part 1 (Foundation) - subsections 1-4
2. Proceed to Part 2 (Configuration) - subsections 5-8
3. Continue with Part 3 (Integration) - subsections 9-10
4. Finish with Part 4 (Standards) - subsections 11-12

**For Experienced Users:**
- Focus on Parts 2-4
- Use Part 1 as reference when needed

**For Team Leads:**
- Review Parts 2 and 4 to standardize team configuration
- Set up project-level settings, commands, and agents

---

## Time Estimates

| Subsection | Time | Difficulty |
|------------|------|------------|
| 1. How Claude Works | 30 min | Beginner |
| 2. Security & Permissions | 45 min | Intermediate |
| 3. Memory & Context | 30 min | Beginner |
| 4. CLI Reference | 30 min | Beginner |
| 5. Project Configuration | 60 min | Intermediate |
| 6. Slash Commands | 45 min | Intermediate |
| 7. Agents & Sub-agents | 60 min | Advanced |
| 8. Hooks & Automation | 60 min | Advanced |
| 9. MCP Integration | 45 min | Advanced |
| 10. Git Integration | 30 min | Beginner |
| 11. Standards & Best Practices | 45 min | Intermediate |
| 12. Templates Library | 30 min | Beginner |
| **Total** | **8 hours** | Mixed |

---

## Success Criteria

You're ready to move to Phase 1.4 (Prompt Engineering) when you can:

✅ **Explain** Claude Code's architecture and workflow
✅ **Configure** permissions for different environments (dev, prod)
✅ **Create** a CLAUDE.md file with project standards
✅ **Use** essential CLI commands and keyboard shortcuts
✅ **Set up** project configuration (.claude/settings.json)
✅ **Write** custom slash commands for common tasks
✅ **Define** specialized agents for specific workflows
✅ **Implement** hooks for compliance and automation
✅ **Install** MCP servers for external integrations
✅ **Understand** banking IT git policy (manual operations)
✅ **Apply** PySpark coding standards
✅ **Use** templates for common data engineering tasks

---

## Quick Reference

**Essential Files:**
```
your-project/
├── .claude/
│   ├── settings.json           # Team configuration
│   ├── settings.local.json     # Personal preferences
│   ├── CLAUDE.md               # Project standards
│   ├── commands/               # Custom slash commands
│   │   ├── compliance-check.md
│   │   ├── security-review.md
│   │   └── pipeline-docs.md
│   ├── scripts/                # Hook scripts
│   │   ├── audit_log.py
│   │   ├── detect_secrets.py
│   │   └── pci_compliance.py
│   └── mcp.json                # MCP server configuration
└── CLAUDE.md                   # Alternative location
```

**Essential Commands:**
```bash
claude                          # Start Claude Code
claude --plan                   # Read-only mode
claude -p "query"               # Print mode
/help                           # Show available commands
/config                         # View configuration
/clear                          # Clear conversation
```

---

## Next Steps

**Ready to dive in?**

👉 **[Start with Subsection 1: How Claude Code Works](./03-01-how-claude-works.md)**

**Need help?**
- Type `/help` in Claude Code
- Check [Troubleshooting FAQ](../reference/troubleshooting-faq.md)
- Ask your team lead

---

## Summary

This Phase 1.3 section is split into 12 focused subsections for easier learning:

**Part 1 (Foundation):** Understand the basics of Claude Code
**Part 2 (Configuration):** Configure projects, commands, agents, and hooks
**Part 3 (Integration):** Connect to external systems and git
**Part 4 (Standards):** Apply best practices and use templates

**Estimated Time:** 8 hours total (can be split across multiple sessions)

**Next:** After completing all 12 subsections, continue to [Phase 1.4: Prompt Engineering](./04-prompt-engineering-data-engineering.md)

---

**Last Updated:** 2025-10-23
**Version:** 1.0
