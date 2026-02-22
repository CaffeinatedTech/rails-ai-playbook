# Quality Enforcement in OpenCode

> This document explains how quality enforcement works in OpenCode compared to Claude Code hooks.

---

## Overview

OpenCode does not use Claude Code's JSON-based hook system. Instead, it provides alternative mechanisms for quality enforcement:

1. **Formatters** - Run automatically before saving files
2. **AGENTS.md Instructions** - Guide OpenCode's behavior
3. **Custom Agents** - Wrap behavior with enforcement
4. **Traditional Git Hooks** - Work with any tool

---

## Formatters (Recommended)

Configure formatters in your project's `opencode.json` to automatically run linters:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "formatter": {
    "rubocop": {
      "command": ["bundle", "exec", "rubocop", "--autocorrect"],
      "extensions": [".rb"]
    },
    "eslint": {
      "command": ["npx", "eslint", "--fix"],
      "extensions": [".ts", ".tsx", ".js", ".jsx"]
    },
    "prettier": {
      "command": ["npx", "prettier", "--write"],
      "extensions": [".json", ".md", ".yml"]
    }
  }
}
```

Formatters run automatically when OpenCode saves files.

---

## AGENTS.md Instructions

Add quality guidelines to your project's `AGENTS.md`:

```markdown
## Quality Guidelines

Before completing any task:
- Run linters: `bundle exec rubocop` for Ruby, `npx eslint` for TypeScript
- Ensure tests pass: `bin/rails test`
- Check for hardcoded values that should use environment variables
```

---

## Custom Agents

Create custom agents with enforced behavior in `.opencode/agents/`:

```markdown
---
name: quality-agent
description: Agent with enforced quality checks
---

You are a quality-focused agent. Before completing any task:
1. Run `bundle exec rubocop` 
2. Fix any linting errors
3. Run tests and ensure they pass
```

---

## Traditional Git Hooks

For guaranteed enforcement regardless of AI tool, create a traditional git hook:

```bash
# .git/hooks/post-commit
#!/bin/bash
# Run your quality checks here
bundle exec rubocop
```

Make it executable: `chmod +x .git/hooks/post-commit`

---

## Reference: Claude Code Hook Script

The `post-commit-audit.sh` script in this directory was designed for Claude Code's hook system. It does not work with OpenCode directly. You can:

1. Use it as a reference for what checks to run
2. Adapt it as a traditional git hook (remove the Claude Code JSON output)
3. Use the formatter approach above instead

---

## Browser Automation

Enable OpenCode's built-in browser automation for testing:

```json
{
  "browser": true
}
```

Or set `OPENCODE_ENABLE_BROWSER=true` environment variable.

Install browsers: `npx playwright install chromium`
