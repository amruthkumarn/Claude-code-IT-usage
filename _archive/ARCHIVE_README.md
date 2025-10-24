# Archive Directory

**Date Archived:** 2025-10-24

This directory contains the old documentation structure that was replaced by the new 3-phase structure.

---

## What Was Archived

### Old Directory Structure (6 sections)

The original documentation was organized into 6 sections:

1. **01-foundation/** - Introduction and core concepts
2. **02-basics/** - Getting started and CLI reference
3. **03-advanced/** - Advanced features (memory, commands, agents)
4. **04-security/** - Security and hooks
5. **05-integration/** - MCP, git, standards
6. **06-reference/** - Templates and troubleshooting

### Old Documentation Files

- `index.md` - Old homepage
- `CONVERSION_CHECKLIST.md` - JS/TS to Python conversion tracking
- `PHASE_RESTRUCTURE_README.md` - Restructuring plan document
- `RESTRUCTURING_PROGRESS.md` - Progress tracking during conversion

### Other Archived

- `reference/` - Empty reference directory

---

## Why These Were Archived

### Replaced By New 3-Phase Structure

The old 6-section structure was replaced by a more pedagogical 3-phase approach:

**Phase 1: Onboarding (Master Claude Code)**
- Location: `/phase-1-onboarding/`
- Content: 19+ files covering all fundamentals
- Structure: Progressive learning with subsections

**Phase 2: Build (BRD → Code → Deploy)**
- Location: `/phase-2-build/`
- Content: End-to-end workflow from requirements to deployment

**Phase 3: Maintenance (Production Support)**
- Location: `/phase-3-maintenance/`
- Content: Debugging, optimization, incident response

### Benefits of New Structure

✅ **Progressive Learning**: Clear learning path for banking IT data engineers
✅ **Modular Content**: Smaller, focused files (400-800 lines each)
✅ **Banking Focus**: All examples in PySpark with compliance requirements
✅ **Practical Workflow**: Mirrors actual BRD → DDT → Code workflow
✅ **Assessment**: Comprehensive testing at end of Phase 1

---

## Content Mapping

### Where Old Content Went

| Old Section | New Location |
|-------------|--------------|
| 01-foundation/01-introduction-and-installation.md | phase-1-onboarding/01-introduction-getting-started.md + 02-installation.md |
| 01-foundation/02-core-concepts.md | phase-1-onboarding/03-01-how-claude-works.md |
| 02-basics/03-getting-started.md | phase-1-onboarding/01-introduction-getting-started.md |
| 02-basics/04-cli-reference.md | phase-1-onboarding/03-04-cli-reference.md |
| 03-advanced/05-project-configuration.md | phase-1-onboarding/03-05-project-configuration.md |
| 03-advanced/06-memory-management.md | phase-1-onboarding/03-03-memory-context.md |
| 03-advanced/07-slash-commands.md | phase-1-onboarding/03-06-slash-commands.md |
| 03-advanced/08-agents-subagents.md | phase-1-onboarding/03-07-agents-subagents.md |
| 03-advanced/16-prompt-engineering.md | phase-1-onboarding/04-00-prompt-engineering-index.md + subsections |
| 04-security/09-security-compliance.md | phase-1-onboarding/03-02-security-permissions.md |
| 04-security/10-hooks-automation.md | phase-1-onboarding/03-08-hooks-automation.md |
| 05-integration/11-mcp.md | phase-1-onboarding/03-09-mcp-integration.md |
| 05-integration/12-git-integration.md | phase-1-onboarding/03-10-git-integration.md |
| 05-integration/13-standards-best-practices.md | phase-1-onboarding/03-11-standards-best-practices.md |
| 06-reference/14-templates-library.md | phase-1-onboarding/03-12-templates-library.md |
| 06-reference/15-troubleshooting-faq.md | *(Will be in Phase 3: Maintenance)* |

---

## Should You Use Archived Content?

**NO - Use the new 3-phase structure instead.**

The archived content is kept for:
- **Historical reference** - See evolution of documentation
- **Content recovery** - If something was missed during migration
- **Comparison** - Understand improvements in new structure

**For current documentation, always refer to:**
- `/phase-1-onboarding/` - Learning Claude Code
- `/phase-2-build/` - Building data pipelines
- `/phase-3-maintenance/` - Production support

---

## Archive Contents

```
_archive/
├── ARCHIVE_README.md (this file)
├── index.md
├── CONVERSION_CHECKLIST.md
├── PHASE_RESTRUCTURE_README.md
├── RESTRUCTURING_PROGRESS.md
└── old-structure/
    ├── 01-foundation/
    │   ├── 01-introduction-and-installation.md
    │   └── 02-core-concepts.md
    ├── 02-basics/
    │   ├── 03-getting-started.md
    │   └── 04-cli-reference.md
    ├── 03-advanced/
    │   ├── 05-project-configuration.md
    │   ├── 06-memory-management.md
    │   ├── 07-slash-commands.md
    │   ├── 08-agents-subagents.md
    │   └── 16-prompt-engineering.md
    ├── 04-security/
    │   ├── 09-security-compliance.md
    │   └── 10-hooks-automation.md
    ├── 05-integration/
    │   ├── 11-mcp.md
    │   ├── 12-git-integration.md
    │   └── 13-standards-best-practices.md
    ├── 06-reference/
    │   ├── 14-templates-library.md
    │   └── 15-troubleshooting-faq.md
    └── reference/ (empty)
```

---

## Questions?

If you need to reference the old structure or recover content, contact the documentation maintainer.

**Current Documentation:** See `/README.md` in the repository root.

---

**Archived:** 2025-10-24
**Archived By:** Claude Code Documentation Team
**Reason:** Replaced by 3-phase pedagogical structure
