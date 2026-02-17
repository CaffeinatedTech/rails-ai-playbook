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
    render inertia: "Contact/New"
  end

  def create
    ContactMailer.inquiry(
      name: params[:name],
      email: params[:email],
      message: params[:message]
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

### React Page

```jsx
// app/frontend/pages/Contact/New.jsx
import { Head, useForm } from "@inertiajs/react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Textarea } from "@/components/ui/textarea"

export default function ContactNew() {
  const { data, setData, post, processing, errors } = useForm({
    name: "",
    email: "",
    message: "",
  })

  const handleSubmit = (e) => {
    e.preventDefault()
    post("/contact")
  }

  return (
    <>
      <Head title="Contact Us" />

      <div className="max-w-lg mx-auto py-12 px-4">
        <h1 className="text-2xl font-bold mb-6">Contact Us</h1>

        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <Label htmlFor="name">Name</Label>
            <Input
              id="name"
              value={data.name}
              onChange={(e) => setData("name", e.target.value)}
              required
            />
          </div>

          <div>
            <Label htmlFor="email">Email</Label>
            <Input
              id="email"
              type="email"
              value={data.email}
              onChange={(e) => setData("email", e.target.value)}
              required
            />
          </div>

          <div>
            <Label htmlFor="message">Message</Label>
            <Textarea
              id="message"
              rows={5}
              value={data.message}
              onChange={(e) => setData("message", e.target.value)}
              required
            />
          </div>

          <Button type="submit" disabled={processing}>
            Send Message
          </Button>
        </form>
      </div>
    </>
  )
}
```

---

## Option 3: Footer Link Only

Many apps just put a support email in the footer:

```jsx
// In Footer component
<p className="text-sm text-muted-foreground">
  Questions? Email us at{" "}
  <a href="mailto:support@yourapp.com" className="underline">
    support@yourapp.com
  </a>
</p>
```

---

## Spam Prevention

If using a form, consider:

1. **Honeypot field** (hidden field that bots fill out)
2. **Rate limiting** (Rack::Attack)
3. **reCAPTCHA** (if spam becomes a problem)

Simple honeypot:

```jsx
{/* Hidden field - bots will fill this */}
<input
  type="text"
  name="website"
  className="hidden"
  tabIndex={-1}
  autoComplete="off"
/>
```

```ruby
# In controller
def create
  return head :ok if params[:website].present? # Bot detected

  # ... send email
end
```
