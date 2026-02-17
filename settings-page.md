# Settings Page

> User account settings pattern for Rails + Inertia apps.

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
    render inertia: "Settings/Show", props: {
      user: current_user.as_json(only: [:id, :name, :email, :created_at]),
      subscription: subscription_props,
      connected_accounts: connected_accounts_props
    }
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

### React Page

```jsx
// app/frontend/pages/Settings/Show.jsx
import { Head, useForm, usePage } from "@inertiajs/react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"

export default function SettingsShow() {
  const { user, subscription, flash } = usePage().props

  const profileForm = useForm({
    name: user.name || "",
    email: user.email || "",
  })

  const passwordForm = useForm({
    password: "",
    password_confirmation: "",
  })

  const handleProfileSubmit = (e) => {
    e.preventDefault()
    profileForm.patch("/settings")
  }

  const handlePasswordSubmit = (e) => {
    e.preventDefault()
    passwordForm.patch("/settings/update_password", {
      onSuccess: () => passwordForm.reset()
    })
  }

  return (
    <>
      <Head title="Settings" />

      <div className="max-w-2xl mx-auto py-8 px-4 space-y-6">
        <h1 className="text-2xl font-bold">Settings</h1>

        {/* Profile Section */}
        <Card>
          <CardHeader>
            <CardTitle>Profile</CardTitle>
            <CardDescription>Update your account information</CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleProfileSubmit} className="space-y-4">
              <div>
                <Label htmlFor="name">Name</Label>
                <Input
                  id="name"
                  value={profileForm.data.name}
                  onChange={(e) => profileForm.setData("name", e.target.value)}
                />
              </div>
              <div>
                <Label htmlFor="email">Email</Label>
                <Input
                  id="email"
                  type="email"
                  value={profileForm.data.email}
                  onChange={(e) => profileForm.setData("email", e.target.value)}
                />
              </div>
              <Button type="submit" disabled={profileForm.processing}>
                Save Changes
              </Button>
            </form>
          </CardContent>
        </Card>

        {/* Password Section */}
        <Card>
          <CardHeader>
            <CardTitle>Password</CardTitle>
            <CardDescription>Change your password</CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handlePasswordSubmit} className="space-y-4">
              <div>
                <Label htmlFor="password">New Password</Label>
                <Input
                  id="password"
                  type="password"
                  value={passwordForm.data.password}
                  onChange={(e) => passwordForm.setData("password", e.target.value)}
                />
              </div>
              <div>
                <Label htmlFor="password_confirmation">Confirm Password</Label>
                <Input
                  id="password_confirmation"
                  type="password"
                  value={passwordForm.data.password_confirmation}
                  onChange={(e) => passwordForm.setData("password_confirmation", e.target.value)}
                />
              </div>
              <Button type="submit" disabled={passwordForm.processing}>
                Update Password
              </Button>
            </form>
          </CardContent>
        </Card>

        {/* Subscription Section (if applicable) */}
        {subscription && (
          <Card>
            <CardHeader>
              <CardTitle>Subscription</CardTitle>
              <CardDescription>Manage your subscription</CardDescription>
            </CardHeader>
            <CardContent>
              <p>Current plan: <strong>{subscription.plan}</strong></p>
              <p>Status: {subscription.status}</p>
              {/* Add billing portal link via Stripe */}
            </CardContent>
          </Card>
        )}

        {/* Danger Zone */}
        <Card className="border-destructive">
          <CardHeader>
            <CardTitle className="text-destructive">Danger Zone</CardTitle>
            <CardDescription>Irreversible actions</CardDescription>
          </CardHeader>
          <CardContent>
            <Button variant="destructive">Delete Account</Button>
          </CardContent>
        </Card>
      </div>
    </>
  )
}
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
