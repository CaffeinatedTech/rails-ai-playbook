# Hotwire (Turbo + Stimulus)

> Frontend setup patterns for Rails + Hotwire + Tailwind v4.

---

## Stack Overview

| Layer | Technology |
|-------|------------|
| Navigation | Turbo Drive |
| Interactivity | Turbo Frames + Streams |
| JavaScript | Stimulus |
| Styling | Tailwind CSS v4 |
| Components | Custom Tailwind components |

---

## Project Structure

```
app/
├── components/
│   ├── ui/                    # Custom Tailwind components
│   │   ├── button_component.rb
│   │   ├── input_component.rb
│   │   ├── card_component.rb
│   │   └── alert_component.rb
│   └── layouts/
│       └── app_layout.rb
├── controllers/
├── views/
│   ├── pages/
│   │   └── home.html.erb
│   ├── settings/
│   │   └── show.html.erb
│   └── layouts/
│       └── application.html.erb
├── helpers/
│   └── application_helper.rb
└── javascript/
    └── controllers/
        ├── application.js
        └── clipboard_controller.js
```

---

## Turbo Drive

Turbo Drive provides fast page navigation by intercepting link clicks and form submissions, then rendering the new page without a full browser reload.

### Setup (Rails 8)

Rails 8 includes Turbo by default. Verify in your Gemfile:

```ruby
# Gemfile
gem "turbo-rails"
```

```javascript
// app/javascript/application.js
import "@hotwired/turbo-rails"
```

### Application Layout

```erb
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield :title %></title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
    <%= yield :head %>
  </head>
  <body>
    <%= render "shared/header" %>
    <main>
      <%= yield %>
    </main>
  </body>
</html>
```

### Page Titles

```erb
<!-- In any view -->
<% content_for :title, "Dashboard" %>

<!-- Or use a helper -->
<% provide :title, "Dashboard" %>
```

---

## Navigation

### Links

Use standard `<a>` tags. Turbo Drive intercepts same-domain links automatically:

```erb
<nav>
  <%= link_to "Home", root_path %>
  <%= link_to "Pricing", pricing_path %>
  <%= link_to "Dashboard", app_path %>
</nav>
```

### External Links

External links (different domain) work automatically:

```erb
<%= link_to "External", "https://example.com" %>
```

### Disabled Turbo

Opt-out of Turbo for specific links:

```erb
<%= link_to "No Turbo", path, data: { turbo: false } %>
```

---

## Forms

Forms work with Turbo automatically. Use standard Rails form helpers:

### Basic Form

```erb
<%= form_with model: @contact, url: contact_path, data: { turbo: true } do |form| %>
  <div>
    <%= form.label :name %>
    <%= form.text_field :name %>
  </div>
  
  <div>
    <%= form.label :email %>
    <%= form.email_field :email %>
  </div>
  
  <%= form.submit "Send" %>
<% end %>
```

### Form with Validation Errors

```erb
<%= form_with model: @contact, url: contact_path, 
      data: { turbo_stream: true } do |form| %>
  <% if @contact.errors.any? %>
    <div class="bg-red-50 text-red-600 p-4 rounded-lg mb-4">
      <ul>
        <% @contact.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  
  <%= form.text_field :name %>
  <%= form.submit "Submit" %>
<% end %>
```

---

## Turbo Frames

Turbo Frames scope navigation to a portion of the page. Use for:
- Modal dialogs
- Inline editing
- Lazy-loaded content
- Tabbed interfaces

### Basic Frame

```erb
<%= turbo_frame_tag "edit_post", src: edit_post_path(@post) do %>
  <p>Loading...</p>
<% end %>
```

```erb
<!-- In app/views/posts/edit.html.erb -->
<%= turbo_frame_tag "edit_post" do %>
  <%= render "form", post: @post %>
<% end %>
```

### Frame for Modal

```erb
<!-- Trigger -->
<%= link_to "Edit", edit_post_path(@post), 
      data: { turbo_frame: "modal" } %>

<!-- Modal container -->
<%= turbo_frame_tag "modal" do %>
  <div class="fixed inset-0 bg-black/50 flex items-center justify-center">
    <!-- Content loads here -->
  </div>
<% end %>
```

### Frame Actions

Navigate within a frame:

```erb
<%= link_to "Next", page_path(@next), 
      data: { turbo_frame: "content_frame" } %>
```

---

## Turbo Streams

Turbo Streams update page sections after form submissions or async actions.

### Controller Response

```ruby
class PostsController < ApplicationController
  def create
    @post = Post.new(post_params)
    
    if @post.save
      respond_to do |format|
        format.turbo_stream
        format.html { redirect_to posts_path, notice: "Post created" }
      end
    else
      render :new, status: :unprocessable_entity
    end
  end
end
```

### Stream Views

```erb
<!-- app/views/posts/create.turbo_stream.erb -->
<%= turbo_stream.append "posts", partial: "post", 
      locals: { post: @post } %>

<!-- Or replace the form with a success message -->
<%= turbo_stream.replace "new_post", partial: "success" %>
```

### Stream Actions

| Action | Description |
|--------|-------------|
| `append` | Add to end of target |
| `prepend` | Add to beginning of target |
| `replace` | Replace target element |
| `update` | Replace only content |
| `remove` | Remove element |
| `before` | Insert before target |
| `after` | Insert after target |

### Example: Flash Messages

```ruby
# Controller
def update
  if @resource.update(resource_params)
    redirect_to settings_path, notice: "Updated successfully"
  else
    render :edit, status: :unprocessable_entity
  end
end
```

```erb
<!-- app/views/shared/_flash.html.erb -->
<% if notice %>
  <div id="flash" class="fixed top-4 left-1/2 -translate-x-1/2 z-50">
    <div class="bg-green-50 border border-green-200 text-green-700 px-4 py-3 rounded-lg shadow-lg">
      <%= notice %>
    </div>
  </div>
<% end %>

<% if alert %>
  <div id="flash" class="fixed top-4 left-1/2 -translate-x-1/2 z-50">
    <div class="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded-lg shadow-lg">
      <%= alert %>
    </div>
  </div>
<% end %>

<!-- Auto-hide with Stimulus -->
<script>
  setTimeout(() => {
    document.getElementById('flash')?.remove()
  }, 3000)
</script>
```

---

## Stimulus

Stimulus adds interactivity to static HTML. Use for:
- Toggling visibility
- Clipboard actions
- Character counters
- Auto-submit on change

### Setup

```javascript
// app/javascript/controllers/index.js
import { application } from "./application"
import HelloController from "./hello_controller"
application.register("hello", HelloController)
```

```javascript
// app/javascript/controllers/application.js
import { Application } from "@hotwired/stimulus"

const application = Application.start()
application.debug = false
window.Stimulus   = application

export { application }
```

### Controller Example: Clipboard

```javascript
// app/javascript/controllers/clipboard_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "source" ]

  copy() {
    navigator.clipboard.writeText(this.sourceTarget.value)
    
    // Show feedback
    const button = this.element.querySelector('button')
    const originalText = button.textContent
    button.textContent = "Copied!"
    setTimeout(() => button.textContent = originalText, 2000)
  }
}
```

```erb
<!-- Usage -->
<div data-controller="clipboard">
  <input data-clipboard-target="source" value="Copy me" readonly>
  <button data-action="clipboard#copy">Copy</button>
</div>
```

### Controller Example: Toggle

```javascript
// app/javascript/controllers/toggle_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "content" ]

  toggle() {
    this.contentTarget.classList.toggle("hidden")
  }
}
```

```erb
<div data-controller="toggle">
  <button data-action="toggle#toggle">Show/Hide</button>
  <div data-toggle-target="content" class="hidden">
    Content here
  </div>
</div>
```

### Controller Example: Auto-submit

```javascript
// app/javascript/controllers/auto_submit_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  submit() {
    this.element.requestSubmit()
  }
}
```

```erb
<%= form_with model: @user, data: { controller: "auto_submit", 
      action: "change->auto_submit#submit" } do |form| %>
  <%= form.check_box :notifications %>
<% end %>
```

---

## Custom Tailwind Components

Build your own components using ViewComponent or plain ERB partials.

### Button Component

```erb
<!-- app/components/ui/button_component.rb -->
class Ui::ButtonComponent < ViewComponent::Base
  attr_reader :variant, :size
  
  def initialize(variant: :primary, size: :md, **attrs)
    @variant = variant
    @size = size
    @attrs = attrs
  end
  
  def variant_classes
    case variant
    when :primary
      "bg-primary text-primary-foreground hover:bg-primary/90"
    when :secondary
      "bg-secondary text-secondary-foreground hover:bg-secondary/80"
    when :outline
      "border border-input bg-background hover:bg-accent hover:text-accent-foreground"
    when :destructive
      "bg-destructive text-destructive-foreground hover:bg-destructive/90"
    when :ghost
      "hover:bg-accent hover:text-accent-foreground"
    else
      "bg-primary text-primary-foreground hover:bg-primary/90"
    end
  end
  
  def size_classes
    case size
    when :sm then "h-9 px-3 text-sm"
    when :lg then "h-11 px-8 text-base"
    when :icon then "h-10 w-10"
    else "h-10 px-4 py-2 text-sm"
    end
  end
end
```

```erb
<!-- app/components/ui/button_component.html.erb -->
<%= button_tag(@attrs.merge(class: [variant_classes, size_classes, 
      "inline-flex items-center justify-center rounded-md font-medium 
      ring-offset-background transition-colors focus-visible:outline-none 
      focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 
      disabled:pointer-events-none disabled:opacity-50"])) do %>
  <%= content %>
<% end %>
```

```erb
<!-- Usage -->
<%= render Ui::ButtonComponent.new(variant: :primary) do %>
  Submit
<% end %>

<%= render Ui::ButtonComponent.new(variant: :outline, size: :sm) do %>
  Cancel
<% end %>
```

### Input Component

```erb
<!-- app/components/ui/input_component.rb -->
class Ui::InputComponent < ViewComponent::Base
  attr_reader :field, :label, :error
  
  def initialize(form:, field:, label: nil, error: nil, **attrs)
    @form = form
    @field = field
    @label = label
    @error = error
    @attrs = attrs
  end
  
  def error?
    @error.present? || (@form.object.respond_to?(:errors) && 
                        @form.object.errors[@field].present?)
  end
  
  def error_message
    @error || @form.object.errors[@field].first
  end
end
```

```erb
<!-- app/components/ui/input_component.html.erb -->
<div class="space-y-2">
  <% if @label %>
    <%= @form.label @field, @label, class: "text-sm font-medium leading-none" %>
  <% end %>
  <%= @form.text_field @field, @attrs.merge(
    class: ["flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm",
            "ring-offset-background focus-visible:outline-none focus-visible:ring-2",
            "focus-visible:ring-ring focus-visible:ring-offset-2",
            "disabled:opacity-50 disabled:cursor-not-allowed",
            error? ? "border-destructive focus-visible:ring-destructive" : 
                     "border-input focus-visible:border-ring"]
  ) %>
  <% if error? %>
    <p class="text-sm text-destructive"><%= error_message %></p>
  <% end %>
</div>
```

```erb
<!-- Usage -->
<%= render Ui::InputComponent.new(form: form, field: :name, label: "Name" %>
```

### Card Component

```erb
<!-- app/components/ui/card_component.rb -->
class Ui::CardComponent < ViewComponent::Base
end
```

```erb
<!-- app/components/ui/card_component.html.erb -->
<div class="rounded-lg border bg-card text-card-foreground shadow-sm">
  <%= content %>
</div>
```

```erb
<!-- Usage -->
<%= render Ui::CardComponent.new do %>
  <div class="p-6">
    <h3 class="text-2xl font-semibold">Card Title</h3>
    <p class="text-muted-foreground">Card content here</p>
  </div>
<% end %>
```

### Alert Component

```erb
<!-- app/views/shared/_alert.html.erb -->
<div class="<%= alert_class %> p-4 rounded-lg" role="alert">
  <% if title %>
    <h3 class="font-semibold"><%= title %></h3>
  <% end %>
  <p><%= message %></p>
</div>
```

```erb
<!-- Usage -->
<%= render "shared/alert", 
      alert_class: "bg-green-50 text-green-800 border-green-200",
      title: "Success",
      message: "Your changes have been saved." %>
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

## Icons

Use inline SVG icons or create a helper:

```erb
<!-- app/helpers/icon_helper.rb -->
module IconHelper
  def icon(name, **opts)
    svg = case name
    when :check
      '<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="m4.5 12.75 6 6 9-13.5" /></svg>'
    when :x
      '<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" /></svg>'
    when :settings
      '<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M9.594 3.94c.09-.542.56-.94 1.11-.94h2.593c.55 0 1.02.398 1.11.94l.213 1.281c.063.374.313.686.645.87.074.04.147.083.22.127.325.196.72.257 1.075.124l1.217-.456a1.125 1.125 0 0 1 1.37.49l1.296 2.247a1.125 1.125 0 0 1-.26 1.431l-1.003.827c-.293.241-.438.613-.43.992a7.723 7.723 0 0 1 0 .255c-.008.378.137.75.43.991l1.004.827c.424.35.534.955.26 1.43l-1.298 2.247a1.125 1.125 0 0 1-1.369.491l-1.217-.456c-.355-.133-.75-.072-1.076.124a6.47 6.47 0 0 1-.22.128c-.331.183-.581.495-.644.869l-.213 1.281c-.09.543-.56.94-1.11.94h-2.594c-.55 0-1.019-.398-1.11-.94l-.213-1.281c-.062-.374-.312-.686-.644-.87a6.52 6.52 0 0 1-.22-.127c-.325-.196-.72-.257-1.076-.124l-1.217.456a1.125 1.125 0 0 1-1.369-.49l-1.297-2.247a1.125 1.125 0 0 1 .26-1.431l1.004-.827c.292-.24.437-.613.43-.991a6.932 6.932 0 0 1 0-.255c.007-.38-.138-.751-.43-.992l-1.004.125 1-.827a1.125 0 0 1-.26-1.43l1.297-2.247a1.125 1.125 0 0 1 1.37-.491l1.216.456c.356.133.751.072 1.076-.124.072-.044.146-.086.22-.128.332-.183.582-.495.644-.869l.214-1.28Z" /><path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z" /></svg>'
    else
      ''
    end
    
    content_tag :span, svg.html_safe, 
          class: ["inline-flex", opts[:class]], 
          title: opts[:title]
  end
end
```

```erb
<!-- Usage -->
<%= icon(:check, class: "w-4 h-4") %>
<%= icon(:settings, class: "w-5 h-5 text-gray-500") %>
```

---

## Page Layouts

### Marketing Layout

```erb
<!-- app/views/layouts/marketing.html.erb -->
<% content_for :body_class, "bg-background" %>

<div class="min-h-screen flex flex-col">
  <header class="border-b">
    <div class="container h-14 flex items-center justify-between">
      <%= link_to "AppName", root_path, class: "font-semibold" %>
      <nav class="flex items-center gap-4">
        <%= link_to "Pricing", pricing_path %>
        <%= link_to "Sign In", new_session_path %>
        <%= render Ui::ButtonComponent.new do %>
          <%= link_to "Get Started", sign_up_path %>
        <% end %>
      </nav>
    </div>
  </header>
  
  <main class="flex-1">
    <%= yield %>
  </main>
  
  <footer class="border-t py-8">
    <div class="container text-sm text-muted-foreground">
      &copy; <%= Date.current.year %> AppName
    </div>
  </footer>
</div>
```

### App Layout

```erb
<!-- app/views/layouts/app.html.erb -->
<div class="min-h-screen flex flex-col">
  <header class="bg-background border-b">
    <div class="container h-14 flex items-center justify-between">
      <%= link_to "AppName", app_path, class: "font-semibold" %>
      <div class="flex items-center gap-4">
        <%= link_to "Dashboard", app_path %>
        <%= link_to "Settings", settings_path %>
        <%= button_to "Logout", session_path, method: :delete, 
              class: "text-sm text-muted-foreground hover:text-foreground" %>
      </div>
    </div>
  </header>
  
  <main class="flex-1">
    <%= render "shared/flash" %>
    <%= yield %>
  </main>
</div>
```

---

## Controller → View Pattern

```ruby
class DashboardController < ApplicationController
  def show
    @projects = Current.user.projects.order(created_at: :desc)
    @stats = DashboardService.stats_for(Current.user)
  end
end
```

```erb
<!-- app/views/dashboard/show.html.erb -->
<% content_for :title, "Dashboard" %>

<section class="section-py">
  <div class="container">
    <h1 class="text-3xl font-bold mb-6">Dashboard</h1>
    
    <div class="grid gap-6 md:grid-cols-3">
      <%= render Ui::CardComponent.new do %>
        <div class="p-6">
          <p class="text-sm text-muted-foreground">Total Projects</p>
          <p class="text-2xl font-semibold"><%= @projects.count %></p>
        </div>
      <% end %>
    </div>
  </div>
</section>
```

---

## Frontend Philosophy

> **The frontend is a dumb display layer.** With Hotwire, the server controls everything. Views receive display-ready data and render it. If something needs to think, the server should have thought for it.

- **Server sends display-ready data.** Every instance variable should be ready to render directly. Send `@project.status_label = "Active"` not `@project.status = 1`. Send `@address = "123 Main St"` not raw fields.
- **No data transformation on the client.** No `.map()`, `.filter()`, or `.reduce()` for display logic. If you're transforming data in ERB, that logic belongs in the model or service.
- **Constants live on the server.** Status labels, category labels — defined as model constants. The view never duplicates these mappings.
- **URLs from routes helpers only.** Use `root_url`, `project_path(@project)`, never hardcode paths.
- **Views are simple.** Extract to partials when >50 lines.

---

## Key Rules

1. **Use standard `<a>` and `<form>` tags** - Turbo Drive handles them
2. **Use Turbo Frames for scoped navigation** - modals, inline editing
3. **Use Turbo Streams for partial updates** - form responses, real-time lists
4. **Use Stimulus for interactivity** - toggles, clipboard, auto-submit
5. **Build custom Tailwind components** - no shadcn, no external UI library
6. **Keep views thin** - extract to partials or services when complex
7. **Use ViewComponent** - for reusable, testable UI components

---

## When to Use What

| Need | Solution |
|------|----------|
| Page navigation | Turbo Drive (automatic) |
| Modal dialog | Turbo Frame |
| Inline editing | Turbo Frame |
| Form submission feedback | Turbo Stream |
| Real-time updates | Turbo Stream + Cable |
| Toggle visibility | Stimulus |
| Clipboard action | Stimulus |
| Auto-submit form | Stimulus |
| Character counter | Stimulus |
