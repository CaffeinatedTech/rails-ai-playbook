# Rails AI Playbook

A structured playbook for building Rails apps with [OpenCode](https://opencode.ai). Opinionated setup, living documentation, and automated quality enforcement.

This is the system I use to build Rails applications with OpenCode. It gives OpenCode the context it needs to make good decisions — architecture patterns, code quality rules, testing principles, design system conventions — so I spend less time correcting and more time building.

---

## What's in Here

### Setup & Configuration
| File | What It Does |
|------|-------------|
| [PLAYBOOK.md](PLAYBOOK.md) | Main playbook — OpenCode reads this when creating a new app |
| [project-structure.md](project-structure.md) | Templates for AGENTS.md and all project docs + creation checklist |
| [brand-interview.md](brand-interview.md) | Questions to ask before building — feeds into DESIGN.md |
| [env-template.md](env-template.md) | Environment variable patterns |

### Stack-Specific Guides
| File | What It Does |
|------|-------------|
| [auth.md](auth.md) | Rails 8 authentication + OmniAuth |
| [hotwire.md](hotwire.md) | Hotwire (Turbo + Stimulus) + Tailwind v4 + frontend philosophy |
| [solid-stack.md](solid-stack.md) | Solid Queue/Cache/Cable (single database) |
| [stripe-payments.md](stripe-payments.md) | Stripe CLI workflow + Pay gem |
| [heroku-deploy.md](heroku-deploy.md) | Deployment checklist + Procfile |

### Quality & Standards (copy to each project)
| File | What It Does |
|------|-------------|
| [code-quality.md](code-quality.md) | General code quality rules — controllers, services, models, database |
| [testing-guidelines.md](testing-guidelines.md) | Testing principles — layers, naming, sad paths, mocking, E2E |
| [hooks/](hooks/) | Post-commit quality hooks with linting + design system checks |

### Feature Recipes
| File | What It Does |
|------|-------------|
| [settings-page.md](settings-page.md) | User settings (profile, billing, notifications) |
| [email-verification.md](email-verification.md) | Password reset + email verification |
| [contact-page.md](contact-page.md) | Contact form setup |
| [legal-pages.md](legal-pages.md) | Privacy Policy + Terms of Service |
| [analytics-seo.md](analytics-seo.md) | GA4, Search Console, meta tags, sitemap |
| [logo-generation.md](logo-generation.md) | Logo + favicon generation via AI |

---

## How It Works

### 1. Install

Copy this playbook to your OpenCode config directory:

```bash
git clone https://github.com/One-Man-App-Studio/rails-ai-playbook.git ~/.config/opencode/rails-playbook
```

Then point your global `~/.config/opencode/AGENTS.md` at it:

```markdown
## Project Playbooks

When creating a new **Rails app**, see: `~/.config/opencode/rails-playbook/PLAYBOOK.md`
```

### 2. Create a New App

Tell OpenCode you want to build something. It reads the playbook and:

1. **Interviews you** — project basics, domain model (entities, relationships, business rules), brand identity, technical requirements
2. **Scaffolds project docs** — SCHEMA.md, BUSINESS_RULES.md, DESIGN.md, ARCHITECTURE.md, CODE_QUALITY.md, TESTING.md, and more
3. **Generates the Rails app** — following the stack-specific guides as needed
4. **Configures formatters** — linters run automatically when saving files

### 3. Build Features

As you build, the docs **grow with the project**:
- SCHEMA.md updates with every migration
- BUSINESS_RULES.md captures edge cases as you discover them
- CODE_QUALITY.md gains project-specific rules
- The post-commit hook gets tuned to your design system

The playbook gives you a strong starting point. The project fills in the rest.

---

## The Stack

This playbook is opinionated. It assumes:

| Layer | Technology |
|-------|------------|
| **Server** | Rails 8 + PostgreSQL |
| **Frontend** | Hotwire (Turbo + Stimulus) |
| **Styling** | Tailwind v4 |
| **Auth** | Rails 8 sessions (+ optional OAuth) |
| **Payments** | Stripe via Pay gem |
| **Jobs** | Solid Queue (single DB) |
| **Cache** | Solid Cache (single DB) |
| **Email** | Resend (prod) / letter_opener (dev) |
| **Hosting** | Heroku |

If your stack is different, the structural patterns (living docs, quality hooks, interview-first workflow) still apply — you'd just swap out the stack-specific guides.

---

## Key Ideas

**Interview first.** OpenCode asks about your project before writing code. Domain model, business rules, brand identity, technical requirements. This conversation becomes the documentation that guides the entire build.

**Living documentation.** Docs aren't written once and forgotten. They start with general principles from the playbook and grow with project-specific rules as you build. OpenCode references them on every task.

**Automated quality enforcement.** Configure linters and formatters in opencode.json to catch common violations — linting errors, style issues. OpenCode runs these automatically before saving files.

**Business logic in services, always.** Controllers are thin (authorize, call service, render). Models are thin (validations, scopes, associations). Services are where the real work happens.

---

## Adapting for Your Stack

The playbook is structured so you can swap pieces:

- **Using Hotwire?** See `hotwire.md` for the standard patterns (Turbo + Stimulus + custom Tailwind components)
- **Not using Stripe?** Remove `stripe-payments.md`, swap in your payment provider
- **Not using Heroku?** Replace `heroku-deploy.md` with your deployment target
- **Not using Rails?** The quality principles, testing guidelines, interview workflow, and living docs pattern work with any framework

The core value isn't the specific technologies — it's the system of structured documentation, automated enforcement, and interview-driven project setup.

---

## Additional Setup Requirements

### Browser Automation (for logo generation, testing)

OpenCode has built-in browser automation using Playwright. To enable:

1. Add to your `opencode.json`:

```json
{
  "browser": true
}
```

Or set the environment variable:
```bash
export OPENCODE_ENABLE_BROWSER=true
```

2. Install Playwright browsers:
```bash
npx playwright install chromium
```

This enables 28+ browser automation tools including navigation, clicking, typing, screenshots, and more.

### Persistent Memory (optional)

For persistent memory across sessions, you can use [claude-mem-opencode](https://github.com/mc303/claude-mem-opencode). This is optional and requires additional setup:

1. Install claude-mem v8.5.4 from GitHub (required for worker API):
```bash
git clone https://github.com/thedotmack/claude-mem.git
cd claude-mem
bun install
bun run build
bun link
```

2. Start the claude-mem worker:
```bash
claude-mem worker start
```

3. Use the claude-mem-opencode package in your OpenCode plugins.

Note: This is an advanced feature with complex setup. Only install if you need persistent memory across sessions.

---

## License

MIT
