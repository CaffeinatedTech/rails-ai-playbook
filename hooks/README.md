# Claude Code Hooks

> Quality enforcement hooks that run automatically during development.

---

## Setup

1. Copy hook files to your project's `.claude/hooks/` directory
2. Register in `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/post-commit-audit.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Available Hooks

### post-commit-audit.sh

Runs after every `git commit` command. Checks committed files for:

**Universal checks (work out of the box):**
- ESLint errors on .ts/.tsx files
- TypeScript errors on committed files
- RuboCop offenses on .rb files
- Hardcoded HSL colors in TSX (should use design tokens)
- Raw `<button>` / `<input>` in TSX (should use shadcn components)
- Hardcoded route template literals (should use path props)
- Unscoped `.find()` in controllers (should use scoped queries)
- `.permit!` anywhere (should whitelist attributes)

**Customize per project:**
- Border radius enforcement
- Opacity modifier bans
- Frontend date formatting bans
- Frontend data transformation bans (`.reduce()`, `.map()` for display logic)
- Shadow style enforcement

See the `CUSTOMIZE` comments in the script for where to add project-specific checks.

---

## How It Works

The hook returns failures via `additionalContext`, which Claude sees as part of the commit output. When failures are found, Claude must fix them before moving to the next task.

**Important:** The hook catching issues is only half the story. Claude must actually read the committed files and verify each check — not just acknowledge the output. If the hook passes, do a quick manual review of the checklist items that can't be automated (e.g., "does the server send display-ready data?").
