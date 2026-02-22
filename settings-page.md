# Settings Page

> User account settings pattern for Rails + Hotwire apps.

---

## Interview First

Before building the settings page, ask the user:

1. **What settings do users need to manage?**
   - Profile info (name, email, avatar)?
   - Password change?
   - Email preferences/notifications?
   - Subscription/billing management?
   - Connected accounts (Google, etc.)?
   - Delete account?

2. **Any app-specific settings?**
   - Theme preference (dark/light)?
   - Timezone?
   - Language?
   - Custom app preferences?

3. **Subscription management?**
   - Show current plan?
   - Upgrade/downgrade?
   - Cancel subscription?
   - View invoices?

---

## Standard Settings Structure

### Controller

```ruby
# app/controllers/settings_controller.rb
class SettingsController < ApplicationController
  before_action :authenticate_user!

  def show
    @user = current_user
    @subscription = subscription_props
    @connected_accounts = connected_accounts_props
  end

  def update
    if current_user.update(user_params)
      redirect_to settings_path, notice: "Settings updated"
    else
      redirect_to settings_path, alert: current_user.errors.full_messages.first
    end
  end

  def update_password
    if current_user.update(password_params)
      redirect_to settings_path, notice: "Password updated"
    else
      redirect_to settings_path, alert: "Password update failed"
    end
  end

  def destroy
    current_user.destroy
    redirect_to root_path, notice: "Account deleted"
  end

  private

  def user_params
    params.require(:user).permit(:name, :email)
  end

  def password_params
    params.require(:user).permit(:password, :password_confirmation)
  end

  def subscription_props
    return nil unless current_user.payment_processor

    sub = current_user.payment_processor.subscription
    return nil unless sub

    {
      plan: sub.processor_plan,
      status: sub.status,
      current_period_end: sub.current_period_end,
      cancel_at_period_end: sub.ends_at.present?
    }
  end

  def connected_accounts_props
    # Return connected OAuth accounts if using OmniAuth
    []
  end
end
```

### Routes

```ruby
# config/routes.rb
resource :settings, only: [:show, :update] do
  patch :update_password
  delete :destroy, as: :delete_account
end
```

### ERB View

```erb
<!-- app/views/settings/show.html.erb -->
<% content_for :title, "Settings" %>

<div class="max-w-2xl mx-auto py-8 px-4 space-y-6">
  <h1 class="text-2xl font-bold">Settings</h1>

  <!-- Profile Section -->
  <div class="rounded-lg border bg-card text-card-foreground shadow-sm">
    <div class="flex flex-col space-y-1.5 p-6">
      <h3 class="text-2xl font-semibold leading-none tracking-tight">Profile</h3>
      <p class="text-sm text-muted-foreground">Update your account information</p>
    </div>
    <div class="p-6 pt-0">
      <%= form_with model: @user, url: settings_path, 
            data: { turbo_stream: true }, class: "space-y-4" do |form| %>
        <div class="space-y-2">
          <%= form.label :name, class: "text-sm font-medium leading-none" %>
          <%= form.text_field :name, class: "flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm" %>
        </div>
        <div class="space-y-2">
          <%= form.label :email, class: "text-sm font-medium leading-none" %>
          <%= form.email_field :email, class: "flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm" %>
        </div>
        <%= form.submit "Save Changes", class: "inline-flex items-center justify-center rounded-md text-sm font-medium bg-primary text-primary-foreground hover:bg-primary/90 h-10 px-4 py-2" %>
      <% end %>
    </div>
  </div>

  <!-- Password Section -->
  <div class="rounded-lg border bg-card text-card-foreground shadow-sm">
    <div class="flex flex-col space-y-1.5 p-6">
      <h3 class="text-2xl font-semibold leading-none tracking-tight">Password</h3>
      <p class="text-sm text-muted-foreground">Change your password</p>
    </div>
    <div class="p-6 pt-0">
      <%= form_with url: update_password_settings_path, 
            data: { turbo_stream: true }, class: "space-y-4" do |form| %>
        <div class="space-y-2">
          <%= form.label :password, "New Password", class: "text-sm font-medium leading-none" %>
          <%= form.password_field :password, class: "flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm" %>
        </div>
        <div class="space-y-2">
          <%= form.label :password_confirmation, "Confirm Password", class: "text-sm font-medium leading-none" %>
          <%= form.password_field :password_confirmation, class: "flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm" %>
        </div>
        <%= form.submit "Update Password", class: "inline-flex items-center justify-center rounded-md text-sm font-medium bg-primary text-primary-foreground hover:bg-primary/90 h-10 px-4 py-2" %>
      <% end %>
    </div>
  </div>

  <!-- Subscription Section -->
  <% if @subscription %>
    <div class="rounded-lg border bg-card text-card-foreground shadow-sm">
      <div class="flex flex-col space-y-1.5 p-6">
        <h3 class="text-2xl font-semibold leading-none tracking-tight">Subscription</h3>
        <p class="text-sm text-muted-foreground">Manage your subscription</p>
      </div>
      <div class="p-6 pt-0">
        <p>Current plan: <strong><%= @subscription[:plan] %></strong></p>
        <p>Status: <%= @subscription[:status] %></p>
        <%= link_to "Manage Subscription", billing_portal_path, 
              class: "inline-flex items-center justify-center rounded-md text-sm font-medium bg-primary text-primary-foreground hover:bg-primary/90 h-10 px-4 py-2 mt-4" %>
      </div>
    </div>
  <% end %>

  <!-- Danger Zone -->
  <div class="rounded-lg border border-destructive bg-card text-card-foreground shadow-sm">
    <div class="flex flex-col space-y-1.5 p-6">
      <h3 class="text-2xl font-semibold leading-none tracking-tight text-destructive">Danger Zone</h3>
      <p class="text-sm text-muted-foreground">Irreversible actions</p>
    </div>
    <div class="p-6 pt-0">
      <%= button_to "Delete Account", settings_path, 
            method: :delete,
            data: { turbo_confirm: "Are you sure? This cannot be undone." },
            class: "inline-flex items-center justify-center rounded-md text-sm font-medium bg-destructive text-destructive-foreground hover:bg-destructive/90 h-10 px-4 py-2" %>
    </div>
  </div>
</div>
```

---

## Billing Portal (Stripe)

For subscription management, use Stripe's Customer Portal:

```ruby
# app/controllers/billing_controller.rb
class BillingController < ApplicationController
  def portal
    session = current_user.payment_processor.billing_portal(
      return_url: settings_url
    )
    redirect_to session.url, allow_other_host: true
  end
end

# routes.rb
get "billing/portal", to: "billing#portal"
```

This lets users manage subscriptions, payment methods, and invoices without you building UI.
