# Stripe Payments

> Stripe CLI workflow, Pay gem setup, and webhook handling.

---

## Non-Negotiables

1. **Use Stripe CLI** for creating products, prices, and testing webhooks - never the dashboard
2. **Use Pay gem** for Rails integration - don't build custom Stripe logic
3. **Use PlansService** for centralized plan configuration
4. **Environment-specific price IDs** - different IDs for dev/test/prod
5. **Always use `ENV["X"]`** for Stripe keys - never hardcode

---

## Initial Setup

### 1. Install Stripe CLI

```bash
# macOS
brew install stripe/stripe-cli/stripe

# Login
stripe login
```

### 2. Add Gems

```ruby
# Gemfile
gem 'pay'
gem 'stripe', '~> 15'
```

### 3. Install Pay

```bash
bundle install
bin/rails pay:install:migrations
bin/rails db:migrate
```

### 4. Add to User Model

```ruby
class User < ApplicationRecord
  pay_customer default_payment_processor: :stripe
  # ... rest of model
end
```

---

## Creating Products & Prices (Stripe CLI)

**Always use CLI, not dashboard:**

### Create a Product

```bash
stripe products create \
  --name="Pro Plan" \
  --description="Full access to all features"
```

### Create a Price

```bash
# Monthly subscription
stripe prices create \
  --product="prod_XXX" \
  --unit-amount=2000 \
  --currency=usd \
  --recurring[interval]=month \
  --lookup-key="pro_monthly"

# Annual subscription
stripe prices create \
  --product="prod_XXX" \
  --unit-amount=20000 \
  --currency=usd \
  --recurring[interval]=year \
  --lookup-key="pro_annual"
```

### List Products/Prices

```bash
stripe products list
stripe prices list
```

---

## PlansService Pattern

Centralized plan configuration - single source of truth:

```ruby
# app/services/plans_service.rb

class PlansService
  # Environment-specific price IDs
  PRICE_IDS = {
    test: {
      pro: "price_test_pro",
      max: "price_test_max"
    },
    development: {
      pro: "price_dev_pro",
      max: "price_dev_max"
    },
    production: {
      pro: ENV["STRIPE_PRICE_PRO"],
      max: ENV["STRIPE_PRICE_MAX"]
    }
  }.freeze

  def self.current_price_ids
    env = Rails.env.to_sym
    PRICE_IDS[env] || PRICE_IDS[:development]
  end

  def self.plan_config
    price_ids = current_price_ids

    {
      "free" => {
        name: "Free",
        price: 0,
        stripe_price_id: nil,
        popular: false,
        features: [
          "Basic features",
          "Email support"
        ]
      },
      price_ids[:pro] => {
        name: "Pro",
        price: 20,
        stripe_price_id: price_ids[:pro],
        popular: true,
        features: [
          "All free features",
          "Advanced features",
          "Priority support"
        ]
      },
      price_ids[:max] => {
        name: "Max",
        price: 50,
        stripe_price_id: price_ids[:max],
        popular: false,
        features: [
          "All Pro features",
          "Premium features",
          "Dedicated support"
        ]
      }
    }
  end

  def self.all_plans
    plan_config.map { |id, config| { id: id, **config } }
  end

  def self.paid_plans
    all_plans.reject { |plan| plan[:id] == "free" }
  end

  def self.find_plan(id)
    config = plan_config[id.to_s]
    return nil unless config
    { id: id, **config }
  end

  def self.plan_name(plan_id)
    find_plan(plan_id)&.dig(:name) || "Free"
  end

  def self.for_frontend
    all_plans.map do |plan|
      {
        id: plan[:id],
        name: plan[:name],
        price: plan[:price],
        popular: plan[:popular],
        features: plan[:features]
      }
    end
  end
end
```

---

## Billing Controller

```ruby
# app/controllers/billing_controller.rb

class BillingController < ApplicationController
  allow_unauthenticated_access only: :pricing

  def pricing
    render inertia: 'Pricing', props: {
      plans: PlansService.for_frontend
    }
  end

  def subscribe
    price_id = params[:price_id]
    plan = PlansService.find_plan(price_id)

    return redirect_to pricing_path, alert: "Invalid plan" unless plan

    checkout = Current.user.payment_processor.checkout(
      mode: "subscription",
      line_items: [{ price: price_id, quantity: 1 }],
      success_url: root_url,
      cancel_url: pricing_url
    )

    redirect_to checkout.url, allow_other_host: true
  end

  def portal
    portal_session = Current.user.payment_processor.billing_portal(
      return_url: settings_url
    )
    redirect_to portal_session.url, allow_other_host: true
  end
end
```

---

## Webhook Handling

### Routes

```ruby
# config/routes.rb
post '/webhooks/stripe', to: 'webhooks#stripe'
```

### Controller

```ruby
# app/controllers/webhooks_controller.rb

class WebhooksController < ApplicationController
  allow_unauthenticated_access
  skip_before_action :verify_authenticity_token

  def stripe
    payload = request.body.read
    sig_header = request.env['HTTP_STRIPE_SIGNATURE']

    event = Stripe::Webhook.construct_event(
      payload,
      sig_header,
      ENV['STRIPE_WEBHOOK_SECRET']
    )

    Pay::Webhooks::Stripe.new.call(event)
    head :ok
  rescue JSON::ParserError, Stripe::SignatureVerificationError
    head :bad_request
  end
end
```

### Pay Initializer with Custom Handlers

```ruby
# config/initializers/pay.rb

Pay.setup do |config|
  config.emails.receipt = true
  config.emails.payment_failed = true
end

# Custom webhook handlers
ActiveSupport.on_load(:pay) do
  Pay::Webhooks.delegator.subscribe "stripe.customer.subscription.created" do |event|
    # Handle subscription created
    Rails.logger.info "Subscription created: #{event.data.object.id}"
  end

  Pay::Webhooks.delegator.subscribe "stripe.customer.subscription.deleted" do |event|
    # Handle subscription cancelled
    Rails.logger.info "Subscription deleted: #{event.data.object.id}"
  end
end
```

---

## Testing Webhooks Locally

```bash
# Forward webhooks to local server
stripe listen --forward-to localhost:3000/webhooks/stripe

# In another terminal, trigger test events
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
```

---

## User Helpers

```ruby
# app/models/user.rb

class User < ApplicationRecord
  pay_customer default_payment_processor: :stripe

  def plan_name
    sub = pay_subscriptions.active.last
    return "Free" unless sub
    PlansService.plan_name(sub.processor_plan)
  end

  def subscription_active?
    pay_subscriptions.active.any?
  end

  def free_plan?
    plan_name == "Free"
  end

  def paid_plan?
    !free_plan?
  end
end
```

---

## Frontend Integration

```jsx
// Pricing page
import { usePage, router } from "@inertiajs/react";

function Pricing({ plans }) {
  const { routes, auth } = usePage().props;

  const handleSubscribe = (planId) => {
    if (!auth.authenticated) {
      router.visit(routes.signup + `?plan=${planId}`);
    } else {
      router.get(routes.subscribe, { price_id: planId });
    }
  };

  return (
    <div className="grid md:grid-cols-3 gap-6">
      {plans.map(plan => (
        <PlanCard
          key={plan.id}
          plan={plan}
          onSelect={() => handleSubscribe(plan.id)}
        />
      ))}
    </div>
  );
}
```

---

## Environment Variables

```bash
# .env
STRIPE_PUBLIC_KEY=pk_test_XXX
STRIPE_SECRET_KEY=sk_test_XXX
STRIPE_WEBHOOK_SECRET=whsec_XXX
STRIPE_PRICE_PRO=price_XXX      # Production only
STRIPE_PRICE_MAX=price_XXX      # Production only
```

---

## Stripe CLI Cheatsheet

```bash
# Auth
stripe login
stripe logout

# Products
stripe products create --name="Name" --description="Desc"
stripe products list
stripe products retrieve prod_XXX

# Prices
stripe prices create --product=prod_XXX --unit-amount=2000 --currency=usd --recurring[interval]=month
stripe prices list
stripe prices retrieve price_XXX

# Customers
stripe customers list
stripe customers retrieve cus_XXX

# Subscriptions
stripe subscriptions list
stripe subscriptions retrieve sub_XXX

# Webhooks
stripe listen --forward-to localhost:3000/webhooks/stripe
stripe trigger customer.subscription.created
stripe trigger invoice.payment_succeeded
stripe trigger invoice.payment_failed

# Logs
stripe logs tail
```
