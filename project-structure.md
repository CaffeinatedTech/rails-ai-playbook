# Project Structure Templates

> Templates for AGENTS.md and docs/ folder when creating new projects.

---

## Overview

Every new project should have:

```
my-app/
├── AGENTS.md                    # Brief index, links to docs/
├── README.md                    # Project overview for humans
├── .env.example                 # Required ENV vars
├── .opencode/
│   └── hooks/
│       └── post-commit-audit.sh # Quality enforcement (from playbook)
└── docs/
    ├── SCHEMA.md                # Every table, column, relationship, index
    ├── BUSINESS_RULES.md        # Domain logic, permissions, edge cases
    ├── DESIGN.md                # Brand identity (from interview)
    ├── ARCHITECTURE.md          # Technical decisions
    ├── CODE_QUALITY.md          # Code quality rules (from playbook, grows with project)
    ├── TESTING.md               # Testing principles (from playbook, grows with project)
    ├── PROJECT_SETUP.md         # Local dev, testing, deploy
    └── ROADMAP.md               # Feature priorities
```

---

## AGENTS.md Template

```markdown
# [App Name] - OpenCode Playbook

> Instructions for working on this Rails + Hotwire + Tailwind v4 project.

**Project Docs:**
- [Schema](docs/SCHEMA.md) - Every table, column, relationship, and index
- [Business Rules](docs/BUSINESS_RULES.md) - Domain logic, permissions, edge cases
- [Design System](docs/DESIGN.md) - Colors, typography, components
- [Architecture](docs/ARCHITECTURE.md) - Technical decisions, key flows
- [Code Quality](docs/CODE_QUALITY.md) - Rules for writing clean, maintainable code
- [Testing Guidelines](docs/TESTING.md) - Principles for writing excellent tests
- [Project Setup](docs/PROJECT_SETUP.md) - Local dev, testing, deploy
- [Roadmap](docs/ROADMAP.md) - Feature priorities

---

## Stack

| Layer | Technology |
|-------|------------|
| Server | Rails 8 + PostgreSQL |
| Frontend | Hotwire (Turbo + Stimulus) |
| Styling | Tailwind v4 |
| Auth | Rails 8 sessions [+ Google OAuth] |
| Payments | Stripe via Pay gem |
| Jobs | Solid Queue |
| Email | Resend (prod) / letter_opener (dev) |
| Hosting | Heroku |

---

## Quick Reference

### Commands
\`\`\`bash
bin/dev              # Start dev server
bin/rails test       # Run tests
bin/rails console    # Rails console
\`\`\`

### Key Paths
- Pages: `app/views/pages/`
- Components: `app/components/`
- Services: `app/services/`
- Styles: `app/assets/stylesheets/application.css`

### Conventions
- Internal links: Standard `<a href="...">` (Turbo Drive)
- Routes: Always use Rails route helpers
- Icons: Inline SVG via helper
- Components: Custom Tailwind components (build your own)
- Colors: Tailwind v4 tokens (`bg-background`, not `bg-white`)
- Business logic: Always in `app/services/`, never in controllers or models
- Timestamps: Stored UTC, displayed in user's timezone via `Time.use_zone`

### Frontend Design
When building pages, components, or layouts, use the `/frontend-design` skill to generate distinctive, production-grade UI. Avoids generic AI aesthetics and uses the project's design system (Tailwind v4 + custom components).

---

## Non-Negotiables

1. Small tasks (30-90 min chunks)
2. Tests for behavior changes
3. Never hardcode routes
4. Use ENV for all secrets
5. Update docs when adding features

---

## Available MCP Servers

You can add MCP servers for additional capabilities via `opencode.json`:

```json
{
  "mcp": {
    "exa": {
      "type": "remote",
      "url": "https://mcp.exa.ai"
    }
  }
}
```

For persistent memory, see [claude-mem-opencode](https://github.com/mc303/claude-mem-opencode).

For browser automation, enable built-in browser tools (see PLAYBOOK.md).

---

## docs/PROJECT_SETUP.md Template

```markdown
# Project Setup

> How to run [App Name] locally, test, and deploy.

---

## Prerequisites

- Ruby 3.3+
- Node.js 20+
- PostgreSQL 15+
- Heroku CLI (for deployment)
- Stripe CLI (for webhook testing)

---

## Local Development

### 1. Clone & Install

\`\`\`bash
git clone [repo-url]
cd [app-name]
bundle install
npm install
\`\`\`

### 2. Environment Setup

\`\`\`bash
cp .env.example .env
# Edit .env with your values
\`\`\`

### 3. Database Setup

\`\`\`bash
bin/rails db:create
bin/rails db:migrate
bin/rails db:seed  # Optional
\`\`\`

### 4. Start Dev Server

\`\`\`bash
bin/dev
# App runs at http://localhost:3000
\`\`\`

---

## Testing

\`\`\`bash
# Run all tests
bin/rails test

# Run specific test file
bin/rails test test/models/user_test.rb

# Run specific test
bin/rails test test/models/user_test.rb:42
\`\`\`

---

## Stripe Testing

\`\`\`bash
# Terminal 1: Forward webhooks
stripe listen --forward-to localhost:3000/webhooks/stripe

# Terminal 2: Trigger test events
stripe trigger customer.subscription.created
\`\`\`

---

## Deployment

### Heroku

\`\`\`bash
# Deploy
git push heroku main

# Run migrations (auto via release phase)
# Manual if needed:
heroku run rails db:migrate

# View logs
heroku logs --tail
\`\`\`

### Environment Variables

See `.env.example` for required vars. Set on Heroku:

\`\`\`bash
heroku config:set KEY=value
\`\`\`

---

## Useful Commands

\`\`\`bash
bin/rails console          # Rails console
bin/rails routes           # List routes
bin/rails db:rollback      # Undo last migration
heroku run rails console   # Production console
\`\`\`
```

---

## docs/ARCHITECTURE.md Template

```markdown
# Architecture

> Technical decisions and system design for [App Name].

---

## Data Model

### Core Models

\`\`\`
User
├── has_many :projects
├── has_many :subscriptions (via Pay)
└── attributes: email, name, plan, etc.

Project
├── belongs_to :user
└── attributes: name, description, etc.
\`\`\`

### Database Schema

[Add ERD or key tables]

---

## Key Services

| Service | Purpose |
|---------|---------|
| PlansService | Centralized plan/pricing config |
| [Service] | [Purpose] |

---

## Authentication Flow

1. User signs up (email/password or Google OAuth)
2. Email verification sent
3. Session created via Rails 8 auth
4. JWT not used - cookie-based sessions

---

## Payment Flow

1. User selects plan on pricing page
2. Redirect to Stripe Checkout
3. Webhook receives `subscription.created`
4. User record updated with plan
5. Billing portal for management

---

## Background Jobs

Using Solid Queue (single database).

| Job | Purpose | Priority |
|-----|---------|----------|
| [Job] | [Purpose] | default |

---

## External Integrations

| Service | Purpose | Docs |
|---------|---------|------|
| Stripe | Payments | stripe.com/docs |
| Resend | Email | resend.com/docs |
| [Service] | [Purpose] | [URL] |

---

## Technical Decisions

### Why [Decision]?

[Reasoning]

### Why not [Alternative]?

[Reasoning]
```

---

## docs/ROADMAP.md Template

```markdown
# Roadmap

> Feature priorities and development phases for [App Name].

---

## Current Phase: MVP

**Goal:** [One sentence goal]

**Target:** [Date or milestone]

### Must Have
- [ ] Feature 1
- [ ] Feature 2
- [ ] Feature 3

### Should Have
- [ ] Feature 4
- [ ] Feature 5

### Nice to Have
- [ ] Feature 6

---

## Phase 2: [Name]

**Goal:** [One sentence goal]

### Planned Features
- [ ] Feature A
- [ ] Feature B

---

## Backlog

Ideas for future consideration:
- Idea 1
- Idea 2
- Idea 3

---

## Completed

### Phase 0: Foundation
- [x] Rails 8 setup
- [x] Auth (email + Google)
- [x] Stripe payments
- [x] Heroku deployment
```

---

## README.md Template

```markdown
# [App Name]

> [One-line description]

## Features

- Feature 1
- Feature 2
- Feature 3

## Tech Stack

- Rails 8 + PostgreSQL
- Hotwire (Turbo + Stimulus)
- Tailwind v4
- Stripe payments
- Heroku hosting

## Getting Started

See [docs/PROJECT_SETUP.md](docs/PROJECT_SETUP.md) for setup instructions.

## Documentation

- [Design System](docs/DESIGN.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Roadmap](docs/ROADMAP.md)

## License

[License type]
```

---

## Creation Checklist

When creating a new project:

1. [ ] Create AGENTS.md (brief, links to docs/)
2. [ ] Create README.md
3. [ ] Create docs/ folder
4. [ ] Create docs/SCHEMA.md (from domain model interview)
5. [ ] Create docs/BUSINESS_RULES.md (from domain model interview)
6. [ ] Create docs/DESIGN.md (from brand interview)
7. [ ] Create docs/ARCHITECTURE.md
8. [ ] Create docs/CODE_QUALITY.md (copy from `~/.config/opencode/rails-playbook/code-quality.md`)
9. [ ] Create docs/TESTING.md (copy from `~/.config/opencode/rails-playbook/testing-guidelines.md`)
10. [ ] Create docs/PROJECT_SETUP.md
11. [ ] Create docs/ROADMAP.md
12. [ ] Create .env.example
13. [ ] Create opencode.json with formatter and browser configuration
