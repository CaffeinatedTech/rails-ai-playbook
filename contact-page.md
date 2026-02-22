# Contact Page

> Simple contact form or support link for Rails apps.

---

## Interview First

Ask the user:

1. **What's the support flow?**
   - Email form that sends to your inbox?
   - Link to external tool (Intercom, Crisp, email)?
   - Simple mailto link?

2. **What information do you need?**
   - Just message?
   - Name + email + message?
   - Category/topic selector?

3. **Where should messages go?**
   - Your personal email?
   - Support email (support@domain.com)?
   - Slack channel?

---

## Option 1: Simple Mailto Link

Simplest approach - just link to email:

```jsx
// In footer or help page
<a href="mailto:support@yourapp.com?subject=Support Request">
  Contact Support
</a>
```

---

## Option 2: Contact Form (Sends Email)

### Controller

```ruby
# app/controllers/contact_controller.rb
class ContactController < ApplicationController
  allow_unauthenticated_access

  def new
    @contact = Contact.new
  end

  def create
    ContactMailer.inquiry(
      name: params[:contact][:name],
      email: params[:contact][:email],
      message: params[:contact][:message]
    ).deliver_later

    redirect_to root_path, notice: "Message sent! We'll get back to you soon."
  end
end
```

### Mailer

```ruby
# app/mailers/contact_mailer.rb
class ContactMailer < ApplicationMailer
  def inquiry(name:, email:, message:)
    @name = name
    @email = email
    @message = message

    mail(
      to: ENV["SUPPORT_EMAIL"] || "you@example.com",
      reply_to: email,
      subject: "Contact form: #{name}"
    )
  end
end
```

### Email Template

```erb
<!-- app/views/contact_mailer/inquiry.html.erb -->
<p>New contact form submission:</p>

<p><strong>Name:</strong> <%= @name %></p>
<p><strong>Email:</strong> <%= @email %></p>
<p><strong>Message:</strong></p>
<p><%= simple_format(@message) %></p>
```

### Routes

```ruby
# config/routes.rb
resources :contact, only: [:new, :create]
```

### ERB View

```erb
<!-- app/views/contact/new.html.erb -->
<% content_for :title, "Contact Us" %>

<div class="max-w-lg mx-auto py-12 px-4">
  <h1 class="text-2xl font-bold mb-6">Contact Us</h1>

  <%= form_with model: @contact, url: contact_index_path, 
        data: { turbo_stream: true }, class: "space-y-4" do |form| %>
    <div class="space-y-2">
      <%= form.label :name, class: "text-sm font-medium leading-none" %>
      <%= form.text_field :name, required: true,
            class: "flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm" %>
    </div>

    <div class="space-y-2">
      <%= form.label :email, class: "text-sm font-medium leading-none" %>
      <%= form.email_field :email, required: true,
            class: "flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm" %>
    </div>

    <div class="space-y-2">
      <%= form.label :message, class: "text-sm font-medium leading-none" %>
      <%= form.text_area :message, rows: 5, required: true,
            class: "flex w-full rounded-md border bg-background px-3 py-2 text-sm" %>
    </div>

    <%= form.submit "Send Message",
          class: "inline-flex items-center justify-center rounded-md text-sm font-medium bg-primary text-primary-foreground hover:bg-primary/90 h-10 px-4 py-2" %>
  <% end %>
</div>
```

---

## Option 3: Footer Link Only

Many apps just put a support email in the footer:

```erb
<!-- In Footer partial -->
<p class="text-sm text-muted-foreground">
  Questions? Email us at
  <%= mail_to "support@yourapp.com", "support@yourapp.com", 
        class: "underline hover:text-foreground" %>
</p>
```

---

## Spam Prevention

If using a form, consider:

1. **Honeypot field** (hidden field that bots fill out)
2. **Rate limiting** (Rack::Attack)
3. **reCAPTCHA** (if spam becomes a problem)

Simple honeypot:

```erb
<!-- Hidden field - bots will fill this -->
<div style="display: none;">
  <%= text_field_tag :website, nil, tabindex: -1, autocomplete: "off" %>
</div>
```

```ruby
# In controller
def create
  return head :ok if params[:website].present? # Bot detected

  # ... send email
end
```
