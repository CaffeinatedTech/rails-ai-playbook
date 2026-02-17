# Heroku Deployment

> Checklist and configuration for deploying Rails 8 apps to Heroku.

---

## Prerequisites

```bash
# Install Heroku CLI
brew tap heroku/brew && brew install heroku

# Login
heroku login
```

---

## Create App

```bash
# Create app
heroku create my-app-name

# Add PostgreSQL
heroku addons:create heroku-postgresql:essential-0

# Set stack (if needed)
heroku stack:set heroku-24
```

---

## Required Files

### Procfile

```procfile
web: bundle exec puma -C config/puma.rb
worker: bundle exec rake solid_queue:start
release: bundle exec rails db:migrate
```

### config/puma.rb

```ruby
max_threads_count = ENV.fetch("RAILS_MAX_THREADS") { 5 }
min_threads_count = ENV.fetch("RAILS_MIN_THREADS") { max_threads_count }
threads min_threads_count, max_threads_count

worker_timeout 3600 if ENV.fetch("RAILS_ENV", "development") == "development"

port ENV.fetch("PORT") { 3000 }

environment ENV.fetch("RAILS_ENV") { "development" }

pidfile ENV.fetch("PIDFILE") { "tmp/pids/server.pid" }

workers ENV.fetch("WEB_CONCURRENCY") { 2 }

preload_app!

plugin :solid_queue if ENV.fetch("RAILS_ENV", "development") == "production"
```

---

## Environment Variables

Set all required ENV vars:

```bash
# App basics
heroku config:set APP_NAME="My App"
heroku config:set APP_HOST="my-app-name.herokuapp.com"
heroku config:set APP_URL="https://my-app-name.herokuapp.com"

# Rails
heroku config:set RAILS_ENV=production
heroku config:set RAILS_MASTER_KEY=$(cat config/master.key)
heroku config:set RAILS_SERVE_STATIC_FILES=true

# Stripe
heroku config:set STRIPE_PUBLIC_KEY=pk_live_XXX
heroku config:set STRIPE_SECRET_KEY=sk_live_XXX
heroku config:set STRIPE_WEBHOOK_SECRET=whsec_XXX
heroku config:set STRIPE_PRICE_PRO=price_XXX

# Email (Resend)
heroku config:set RESEND_API_KEY=re_XXX

# OAuth (if using)
heroku config:set GOOGLE_CLIENT_ID=XXX.apps.googleusercontent.com
heroku config:set GOOGLE_CLIENT_SECRET=GOCSPX-XXX

# Analytics (optional)
heroku config:set GA_MEASUREMENT_ID=G-XXX
```

---

## Database Setup

```bash
# Run migrations (also runs on deploy via release phase)
heroku run rails db:migrate

# Seed data (if needed)
heroku run rails db:seed

# Seed Stripe plans
heroku run rails runner db/seeds/pay_plans.rb
```

---

## Deploy

```bash
# Initial deploy
git push heroku main

# Subsequent deploys
git push heroku main

# Deploy specific branch
git push heroku feature-branch:main
```

---

## Stripe Webhook Setup

1. Go to Stripe Dashboard → Developers → Webhooks
2. Add endpoint: `https://my-app-name.herokuapp.com/webhooks/stripe`
3. Select events:
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`
4. Copy signing secret to `STRIPE_WEBHOOK_SECRET`

```bash
heroku config:set STRIPE_WEBHOOK_SECRET=whsec_XXX
```

---

## Custom Domain

```bash
# Add domain
heroku domains:add www.mydomain.com
heroku domains:add mydomain.com

# Get DNS target
heroku domains

# Configure DNS at your registrar:
# - CNAME: www → your-app.herokudns.com
# - ALIAS/ANAME: @ → your-app.herokudns.com
```

---

## SSL

Heroku provides automatic SSL for custom domains on paid dynos.

```bash
# Enable ACM (Automated Certificate Management)
heroku certs:auto:enable
```

---

## Scaling

```bash
# Scale web dynos
heroku ps:scale web=1
heroku ps:scale web=2

# Scale worker dynos
heroku ps:scale worker=1

# Check dyno status
heroku ps
```

---

## Monitoring

```bash
# View logs
heroku logs --tail

# View specific process logs
heroku logs --tail --ps web

# Check app status
heroku ps

# Run console
heroku run rails console

# Run one-off command
heroku run rails runner "puts User.count"
```

---

## Common Issues

### Assets not loading

```bash
heroku config:set RAILS_SERVE_STATIC_FILES=true
```

### Migrations not running

Check Procfile has release phase:
```procfile
release: bundle exec rails db:migrate
```

### Solid Queue not processing

1. Make sure worker dyno is running: `heroku ps:scale worker=1`
2. Check logs: `heroku logs --tail --ps worker`
3. Verify database connection in single-db setup

### Memory issues

```bash
# Check memory usage
heroku logs --tail | grep Memory

# Tune Puma workers
heroku config:set WEB_CONCURRENCY=2
heroku config:set RAILS_MAX_THREADS=5
```

---

## Maintenance

```bash
# Enable maintenance mode
heroku maintenance:on

# Disable maintenance mode
heroku maintenance:off

# Restart dynos
heroku restart

# Database backup
heroku pg:backups:capture
heroku pg:backups:download
```

---

## Pre-Deploy Checklist

- [ ] All ENV vars set on Heroku
- [ ] `config/master.key` value set as `RAILS_MASTER_KEY`
- [ ] Procfile with web, worker, release
- [ ] Database migrations ready
- [ ] Stripe webhook endpoint configured
- [ ] Custom domain DNS configured (if applicable)
- [ ] SSL enabled
- [ ] Tested locally with `RAILS_ENV=production`

---

## Post-Deploy Checklist

- [ ] App loads without errors
- [ ] User can sign up / log in
- [ ] Email verification works
- [ ] Stripe checkout works
- [ ] Webhook events received
- [ ] Background jobs processing
- [ ] Logs show no errors
