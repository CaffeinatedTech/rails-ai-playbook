# Inertia + React + Vite

> Frontend setup patterns for Rails + Inertia + React + Tailwind v4 + shadcn/ui.

---

## Stack Overview

| Layer | Technology |
|-------|------------|
| Build | Vite |
| SPA Bridge | Inertia.js |
| UI Framework | React |
| Styling | Tailwind CSS v4 |
| Components | shadcn/ui |
| Icons | Lucide React |

---

## Project Structure

```
app/frontend/
├── entrypoints/
│   ├── application.js       # Turbo + Stimulus (if needed)
│   ├── application.css      # Tailwind v4 @theme tokens
│   └── inertia.jsx          # Inertia app setup
├── pages/
│   ├── Home.jsx             # Landing page
│   ├── Pricing.jsx          # Pricing page
│   ├── Auth/
│   │   ├── Login.jsx
│   │   └── Signup.jsx
│   └── App/
│       ├── Dashboard.jsx
│       └── Settings.jsx
├── components/
│   ├── ui/                  # shadcn/ui components
│   ├── marketing/           # Landing page components
│   ├── app/                 # App-specific components
│   ├── Flash.jsx            # Flash messages
│   └── ErrorBoundary.jsx    # Error handling
├── layout/
│   └── AppLayout.jsx        # Authenticated shell
└── lib/
    └── utils.js             # cn() helper
```

---

## Tailwind v4 Setup

### application.css

```css
@import "tailwindcss";

@theme {
  --radius: 1rem;

  /* Colors - customize per project */
  --color-background: hsl(0 0% 100%);
  --color-foreground: hsl(222.2 84% 4.9%);

  --color-card: hsl(0 0% 100%);
  --color-card-foreground: hsl(222.2 84% 4.9%);

  --color-primary: hsl(243 75% 59%);
  --color-primary-foreground: hsl(0 0% 100%);

  --color-secondary: hsl(220 14% 96%);
  --color-secondary-foreground: hsl(222.2 84% 4.9%);

  --color-muted: hsl(220 14% 96%);
  --color-muted-foreground: hsl(215 16% 47%);

  --color-accent: hsl(240 100% 97%);
  --color-accent-foreground: hsl(222.2 84% 4.9%);

  --color-destructive: hsl(0 84% 60%);
  --color-destructive-foreground: hsl(210 40% 98%);

  --color-border: hsl(220 13% 91%);
  --color-input: hsl(220 13% 91%);
  --ring: hsl(243 75% 59%);
}

@layer base {
  * { @apply border-border; }
  html, body { height: 100%; }
  body {
    @apply bg-background text-foreground antialiased;
  }
  .container { @apply max-w-[1200px] mx-auto px-6 md:px-8; }
}

.section-py { @apply py-16 md:py-24; }
```

---

## Shared Data (Inertia)

### Rails Controller

```ruby
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  include InertiaRails::Controller

  inertia_share flash: -> { flash.to_hash },
                auth: -> {
                  {
                    user: Current.user ? {
                      id: Current.user.id,
                      email: Current.user.email_address,
                      first_name: Current.user.first_name,
                      last_name: Current.user.last_name,
                      full_name: Current.user.full_name,
                      initials: Current.user.initials,
                      plan_name: Current.user.plan_name,
                      email_verified: Current.user.email_verified?
                    } : nil,
                    authenticated: !!Current.user
                  }
                },
                routes: -> {
                  {
                    home: root_path,
                    login: login_path,
                    signup: sign_up_path,
                    logout: Current.user ? sign_out_path : nil,
                    pricing: pricing_path,
                    app: Current.user ? "/app" : nil,
                    settings: Current.user ? settings_path : nil,
                    billing_portal: Current.user ? "/billing/portal" : nil,
                    subscribe: subscribe_path
                  }
                }
end
```

### React Access

```jsx
import { usePage } from "@inertiajs/react";

function MyComponent() {
  const { auth, routes, flash } = usePage().props;

  // auth.user - current user or null
  // auth.authenticated - boolean
  // routes.* - named routes
  // flash.notice / flash.alert - flash messages

  return (
    <div>
      {auth.authenticated ? (
        <p>Welcome, {auth.user.first_name}!</p>
      ) : (
        <Link href={routes.login}>Log in</Link>
      )}
    </div>
  );
}
```

---

## Navigation

### Internal Links (Inertia)

```jsx
import { Link } from "@inertiajs/react";

// Always use shared routes
function Nav() {
  const { routes } = usePage().props;

  return (
    <nav>
      <Link href={routes.home}>Home</Link>
      <Link href={routes.pricing}>Pricing</Link>
      <Link href={routes.app}>Dashboard</Link>
    </nav>
  );
}
```

### Form Submissions

```jsx
import { useForm } from "@inertiajs/react";

function ContactForm() {
  const { routes } = usePage().props;
  const { data, setData, post, processing, errors } = useForm({
    name: "",
    email: "",
    message: ""
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    post(routes.contact);
  };

  return (
    <form onSubmit={handleSubmit}>
      <Input
        value={data.name}
        onChange={e => setData("name", e.target.value)}
        error={errors.name}
      />
      <Button type="submit" disabled={processing}>
        {processing ? "Sending..." : "Send"}
      </Button>
    </form>
  );
}
```

---

## shadcn/ui Setup

### components.json

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "app/frontend/entrypoints/application.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

### Adding Components

```bash
npx shadcn@latest add button input card alert badge label accordion
```

### Utils Helper

```js
// app/frontend/lib/utils.js
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

---

## Common Patterns

### Page Layout

```jsx
export default function Dashboard() {
  return (
    <section className="section-py">
      <div className="container">
        <h1 className="text-3xl font-bold mb-6">Dashboard</h1>
        {/* Content */}
      </div>
    </section>
  );
}
```

### Flash Messages

```jsx
import { Alert, AlertTitle, AlertDescription } from "@/components/ui/alert";
import { usePage } from "@inertiajs/react";

export default function Flash() {
  const { flash } = usePage().props;

  if (!flash?.alert && !flash?.notice) return null;

  return (
    <div className="fixed top-4 left-1/2 -translate-x-1/2 w-[90%] max-w-sm z-50">
      <Alert variant={flash.alert ? "destructive" : "default"}>
        <AlertTitle>{flash.alert ? "Error" : "Notice"}</AlertTitle>
        <AlertDescription>{flash.alert || flash.notice}</AlertDescription>
      </Alert>
    </div>
  );
}
```

### App Layout

```jsx
import { Link, usePage } from "@inertiajs/react";
import { Button } from "@/components/ui/button";
import Flash from "@/components/Flash";

export default function AppLayout({ children }) {
  const { routes, auth } = usePage().props;

  return (
    <div className="min-h-screen flex flex-col">
      <header className="bg-background border-b">
        <div className="container h-14 flex items-center justify-between">
          <Link href={routes.app} className="font-semibold">
            AppName
          </Link>
          <div className="flex items-center gap-4">
            <Link href={routes.settings}>Settings</Link>
            <form method="POST" action={routes.logout}>
              <input type="hidden" name="_method" value="delete" />
              <input
                type="hidden"
                name="authenticity_token"
                value={document.querySelector('meta[name="csrf-token"]')?.content}
              />
              <Button type="submit" variant="outline">
                Logout
              </Button>
            </form>
          </div>
        </div>
      </header>
      <main className="flex-1">
        <Flash />
        {children}
      </main>
    </div>
  );
}
```

---

## Icons (Lucide React)

```jsx
import { Check, X, Plus, Settings, LogOut } from "lucide-react";

// Always use Lucide React - never inline SVG
<Button>
  <Plus className="mr-2 h-4 w-4" />
  Add Item
</Button>
```

---

## Vite Config

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { fileURLToPath, URL } from "node:url";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./app/frontend", import.meta.url)),
    },
  },
  server: {
    hmr: { overlay: true },
  },
});
```

---

## Controller → Inertia

```ruby
class DashboardController < ApplicationController
  def show
    render inertia: "App/Dashboard", props: {
      projects: Current.user.projects.order(created_at: :desc),
      stats: DashboardService.stats_for(Current.user)
    }
  end
end
```

---

## Frontend Philosophy

> **The frontend is a dumb display layer.** With Inertia, the server controls the page. React components receive display-ready props and render them. If a component needs to think, the server should have thought for it.

- **Server sends display-ready data.** Every prop should be ready to render directly. Send `role_label: "Office/Sales"` not `user_type: "office_sales"`. Send `address: "123 Main St, Denver, CO"` not raw fields for the client to assemble.
- **No data transformation on the client.** No `.reduce()`, `.map()`, or `.filter()` for display logic. No label lookups, no string formatting, no pluralization, no date formatting. If you're writing `===` checks to transform server data, that logic belongs on the server.
- **Constants live on the server.** Role labels, status labels, category labels — defined as model constants and shared via `inertia_share`. The frontend never duplicates these mappings.
- **URLs from server only.** Pass `path`, `edit_path` etc. in props. Never construct URLs on the client with template literals like `` `/things/${id}` ``.
- **Props are explicit hashes.** Never pass raw ActiveRecord objects. Only include fields the frontend actually needs.

---

## Key Rules

1. **Never hardcode routes** - always use `usePage().props.routes`
2. **Use `<Link>` for internal navigation** - not `<a>`
3. **Use shadcn/ui components** - not custom Tailwind divs
4. **Use Lucide React icons** - never inline SVG
5. **Use Tailwind v4 tokens** - `bg-background`, not `bg-white`
6. **Keep pages thin** - extract to components when >100 lines
