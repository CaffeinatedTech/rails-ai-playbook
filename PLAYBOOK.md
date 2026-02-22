# Rails Playbook

> How I like my Rails apps set up. Reference this when creating new projects.

---

## Non-Negotiables

1. **Interview me first** - Ask about the project, brand, and requirements before writing code
2. **Create project docs** - Set up `docs/` folder with DESIGN.md, CODE_QUALITY.md, TESTING.md, etc.
3. **Keep AGENTS.md clean** - Brief index that links to detailed docs
4. **Use .env files** - All secrets via `ENV["X"]`, never hardcode
5. **Stripe CLI only** - Create products/prices via CLI, not dashboard
6. **Single database** - Solid Queue/Cache/Cable all use primary DB
7. **Small tasks** - Prefer 30-90 minute chunks, suggest splits if scope creeps

---

## Stack

| Layer | Technology |
|-------|------------|
| **Server** | Rails 8 + PostgreSQL |
| **Auth** | Rails 8 sessions + Google OAuth (optional) |
| **Frontend** | Hotwire (Turbo + Stimulus) |
| **Styling** | Tailwind v4 |
| **Payments** | Stripe via Pay gem |
| **Jobs** | Solid Queue (single DB) |
| **Cache** | Solid Cache (single DB) |
| **WebSockets** | Solid Cable (single DB) |
| **Email** | Resend (prod) / letter_opener_web (dev) |
| **Hosting** | Heroku |

---

## When Creating a New App

### Step 1: Interview Me

Before writing any code, ask me about:

**Project basics:**
- What does this app do? (one sentence)
- Who is the target user?
- What's the MVP scope?

**Domain model:**
- What are the core entities? How do they relate?
- Who are the user types? What can each one do?
- What are the key business rules and constraints?
- What are the important edge cases?

**Brand identity:** (see [brand-interview.md](brand-interview.md))
- Colors, typography, component style
- Tone of voice
- Reference sites I like

**Technical requirements:**
- Need Google OAuth?
- Need Stripe payments?
- Any specific integrations?

### Step 2: Create Project Structure

After the interview, generate:

```
my-app/
├── AGENTS.md                    # Brief index (see project-structure.md)
├── README.md                    # Project overview
├── .env.example                 # Required ENV vars
├── .opencode/
│   └── hooks/
│       └── post-commit-audit.sh # Quality enforcement (from playbook)
└── docs/
    ├── SCHEMA.md                # Every table, column, relationship, index
    ├── BUSINESS_RULES.md        # Domain logic, permissions, edge cases
    ├── DESIGN.md                # Brand identity from interview
    ├── ARCHITECTURE.md          # Technical decisions
    ├── CODE_QUALITY.md          # Code quality rules (from playbook)
    ├── TESTING.md               # Testing principles (from playbook)
    ├── PROJECT_SETUP.md         # Local dev, testing, deploy
    └── ROADMAP.md               # Feature priorities
```

### Step 3: Generate Rails App

Use my template or set up manually following these docs:
- [auth.md](auth.md) - Rails 8 authentication + OmniAuth
- [solid-stack.md](solid-stack.md) - Database + Solid* config
- [stripe-payments.md](stripe-payments.md) - Payments setup
- [hotwire.md](hotwire.md) - Frontend setup (Turbo + Stimulus)
- [heroku-deploy.md](heroku-deploy.md) - Deployment config
- [env-template.md](env-template.md) - Environment variables

### Step 4: Add Common Features

After the core app is running, add these as needed:
- [settings-page.md](settings-page.md) - User settings (profile, billing, notifications)
- [email-verification.md](email-verification.md) - Password reset + email verification
- [contact-page.md](contact-page.md) - Contact form or simple mailto
- [legal-pages.md](legal-pages.md) - Privacy Policy + Terms of Service
- [analytics-seo.md](analytics-seo.md) - GA4, Search Console, meta tags, sitemap
- [logo-generation.md](logo-generation.md) - Generate logo + favicons via AI

---

## Conventions

### Rails

- Controllers thin; business logic in `app/services/`
- **LLM integration:** Use `ruby_llm` gem (unified interface for OpenAI, Anthropic, Gemini, OpenRouter). Only use `ruby-openai` if explicitly requested.
- Strong params only; never `.permit!`
- Jobs idempotent; use Solid Queue
- Public actions explicit: `allow_unauthenticated_access`
- Use `ENV["X"]` for all configuration

### Hotwire (Turbo + Stimulus)

- Views in `app/views/`
- Components in `app/components/`
- Internal links use standard `<a href="...">` (Turbo Drive)
- **Never hardcode routes** - use Rails route helpers
- Build custom Tailwind components (no external UI library)

### Frontend Design

When building pages, components, or layouts, use the `/frontend-design` skill to generate distinctive, production-grade UI. This avoids generic AI aesthetics and produces polished code using the project's design system (Tailwind v4 + custom components).

Use it for:
- New pages (landing, dashboard, settings, etc.)
- Complex components (data tables, forms, navigation)
- Layout shells (app layout, marketing layout)
- Any UI that should look polished and distinctive

### Tailwind v4

- Theme tokens in `@theme` block (application.css)
- Use token utilities: `bg-background`, `text-foreground`, `border-border`
- Layout: `.container` for width, `.section-py` for vertical rhythm

---

## Key Docs

| Doc | Purpose |
|-----|---------|
| [auth.md](auth.md) | Rails 8 auth generator + OmniAuth patterns |
| [brand-interview.md](brand-interview.md) | Questions to ask about design/identity |
| [solid-stack.md](solid-stack.md) | Single-database Solid Queue/Cache/Cable |
| [stripe-payments.md](stripe-payments.md) | Stripe CLI workflow + Pay gem |
| [hotwire.md](hotwire.md) | Turbo + Stimulus + custom Tailwind components + frontend philosophy |
| [heroku-deploy.md](heroku-deploy.md) | Deployment checklist + Procfile |
| [env-template.md](env-template.md) | Required environment variables |
| [project-structure.md](project-structure.md) | Templates for project docs |
| [code-quality.md](code-quality.md) | General code quality rules (copy to project docs/) |
| [testing-guidelines.md](testing-guidelines.md) | General testing principles (copy to project docs/) |
| [hooks/](hooks/) | Quality enforcement reference (manual setup) |
| [settings-page.md](settings-page.md) | User settings with profile, billing, notifications |
| [contact-page.md](contact-page.md) | Contact form or mailto link setup |
| [legal-pages.md](legal-pages.md) | Privacy Policy & Terms of Service templates |
| [logo-generation.md](logo-generation.md) | Logo + favicon generation via AI tools |
| [analytics-seo.md](analytics-seo.md) | GA4, Search Console, meta tags, sitemap |
| [email-verification.md](email-verification.md) | Password reset + email verification flows |

---

## Testing

See [testing-guidelines.md](testing-guidelines.md) for full principles. Quick reference:

- Runner: `bin/rails test`
- Add meaningful assertions (not just status codes)
- Happy path + edge case for each feature
- Stub external services (Stripe, APIs)
- Test at the lowest possible layer

---

## Review Checklist

Before completing any task:

- [ ] Tests added/updated, `bin/rails test` passes
- [ ] Standard `<a>` links for internal navigation (Turbo Drive)
- [ ] Custom Tailwind components for controls & cards
- [ ] Tailwind v4 tokens (not hardcoded colors)
- [ ] No secrets in code (use ENV)
- [ ] Public endpoints have `allow_unauthenticated_access`
