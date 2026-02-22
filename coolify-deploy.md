# Coolify Deployment

> Deploy Rails 8 + SQLite apps on Coolify v4 using a private Docker registry.

---

## Overview

- Docker image built locally and pushed to your private registry
- Coolify pulls the image and deploys it
- SQLite database stored in a mounted volume for persistence
- SSL handled by Coolify's built-in Traefik

---

## Prerequisites

- Coolify v4 installed and running
- Private Docker registry accessible from your Coolify server
- Rails app configured with SQLite (see [solid-stack.md](solid-stack.md))

---

## Step 1: Dockerfile

Create a `Dockerfile` in your Rails project root:

```dockerfile
# Stage 1: Dependencies
FROM ruby:3.3-slim AS deps

RUN apt-get update -qq && \
    apt-get install -y build-essential libpq-dev libsqlite3-dev nodejs npm && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY Gemfile Gemfile.lock ./
RUN bundle install --jobs 4 --retry 3

COPY package.json package-lock.json ./
RUN npm install --jobs 4

# Stage 2: Build
FROM ruby:3.3-slim AS builder

RUN apt-get update -qq && \
    apt-get install -y build-essential libpq-dev libsqlite3-dev nodejs npm && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=deps /usr/local/bundle /usr/local/bundle
COPY --from=deps /app/node_modules ./node_modules

COPY . .
RUN bundle exec rails assets:precompile || true

# Stage 3: Runtime
FROM ruby:3.3-slim

RUN apt-get update -qq && \
    apt-get install -y libsqlite3-0 nodejs && \
    rm -rf /var/lib/apt/lists/*

RUN useradd --create-home appuser

WORKDIR /app

COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/public ./public
COPY --from=builder /app/db ./db
COPY --from=builder --chown=appuser:appuser . .

USER appuser

EXPOSE 3000

CMD ["bin/rails", "server", "-b", "0.0.0.0"]
```

---

## Step 2: Build and Push

Build the image and push to your private registry:

```bash
# Build
docker build -t my-registry.com/my-app:latest .

# Push to registry
docker push my-registry.com/my-app:latest
```

---

## Step 3: Coolify App Setup

### Create App

1. In Coolify dashboard, create a new application
2. Select **Docker** as the type
3. Configure your private registry:
   - Registry URL: `my-registry.com`
   - Image: `my-app:latest`

### Environment Variables

Set these in Coolify's environment variables section:

```bash
# Required
RAILS_ENV=production
RAILS_SERVE_STATIC_FILES=true
SECRET_KEY_BASE=$(rails secret)

# App-specific
APP_NAME="My App"
APP_URL="https://yourdomain.com"

# Stripe (if using)
STRIPE_PUBLIC_KEY=pk_live_XXX
STRIPE_SECRET_KEY=sk_live_XXX
STRIPE_WEBHOOK_SECRET=whsec_XXX

# Email (if using Resend)
RESEND_API_KEY=re_XXX

# OAuth (if using)
GOOGLE_CLIENT_ID=XXX.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-XXX

# Analytics (optional)
GA_MEASUREMENT_ID=G-XXX
```

**Note:** No `DATABASE_URL` is needed — SQLite uses file-based storage.

### Volume Mounts

Configure these mounts in Coolify to persist data across deployments:

| Container Path | Host Path | Purpose |
|----------------|-----------|---------|
| `/app/db` | `/coolify/volumes/my-app/db` | SQLite database |
| `/app/storage` | `/coolify/volumes/my-app/storage` | File uploads |

Create these directories on your Coolify server first:
```bash
mkdir -p /coolify/volumes/my-app/db
mkdir -p /coolify/volumes/my-app/storage
```

---

## Step 4: Run Migrations

### Option A: Pre-deployment Command

In Coolify UI, go to your app's **Settings** → **Pre-deployment** and add:

```bash
bundle exec rails db:migrate
```

### Option B: Entrypoint Script

Create `entrypoint.sh` in your project root:

```bash
#!/bin/bash
set -e

echo "Running migrations..."
bundle exec rails db:migrate

echo "Starting server..."
exec "$@"
```

Update your Dockerfile to use this entrypoint:

```dockerfile
COPY entrypoint.sh ./
RUN chmod +x entrypoint.sh

EXPOSE 3000
ENTRYPOINT ["./entrypoint.sh"]
CMD ["bin/rails", "server", "-b", "0.0.0.0"]
```

---

## Step 5: SSL

Coolify's built-in Traefik handles SSL automatically:

1. In Coolify, go to your app **Settings** → **Domains**
2. Add your domain (e.g., `myapp.yourdomain.com`)
3. Enable "HTTPS" toggle
4. SSL certificate is automatically provisioned via Let's Encrypt

---

## Step 6: Automated Backups to S3

### Backup Script

Create `bin/backup.sh`:

```bash
#!/bin/bash
set -e

S3_BUCKET="your-backup-bucket"
DB_PATH="/app/db/production.sqlite3"
BACKUP_DIR="/tmp/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.sqlite3"

mkdir -p "$BACKUP_DIR"

echo "Creating backup..."
cp "$DB_PATH" "$BACKUP_DIR/$BACKUP_FILE"

echo "Uploading to S3..."
aws s3 cp "$BACKUP_DIR/$BACKUP_FILE" "s3://${S3_BUCKET}/rails-apps/my-app/${BACKUP_FILE}"

echo "Cleaning up local backups older than 7 days..."
find "$BACKUP_DIR" -type f -mtime +7 -delete

echo "Backup complete: ${BACKUP_FILE}"
```

Make it executable:
```bash
chmod +x bin/backup.sh
```

### Add AWS Credentials

In Coolify, add these environment variables:

```bash
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_DEFAULT_REGION=us-east-1
```

### Scheduling

In Coolify, create a **Scheduled Task**:

- Type: Application
- Command: `bin/backup.sh`
- Cron: `0 3 * * *` (daily at 3 AM)

### Restore Procedure

```bash
# Download from S3
aws s3 cp s3://your-backup-bucket/rails-apps/my-app/backup_20240115_030000.sqlite3 /tmp/restore.sqlite3

# Stop the app in Coolify

# Restore
cp /tmp/restore.sqlite3 /app/db/production.sqlite3

# Start the app in Coolify

# Verify
bundle exec rails runner "puts User.count"
```

---

## Step 7: Monitoring

### Logs

View logs in Coolify UI: **App** → **Logs**

### SSH into Container

Coolify UI: **App** → **Deployment** → **Console**

Useful commands:
```bash
# Check database
rails dbconsole

# Check disk space
df -h

# Check processes
ps aux

# Check memory
free -h
```

### Health Check

Add a simple health check endpoint:

```ruby
# config/routes.rb
get "/up" => "rails/health#show", as: :health_check
```

This returns `200 OK` when the app is healthy.

---

## Troubleshooting

### App won't start

1. Check logs in Coolify
2. Verify volume mounts are correct
3. Ensure `SECRET_KEY_BASE` is set
4. Check that `/app/db` directory exists and is writable

### Database errors

1. Verify the volume mount exists on the host
2. Check file permissions: `ls -la /coolify/volumes/my-app/db`
3. Ensure SQLite file exists: `ls -la /app/db/*.sqlite3`

### Out of disk space

1. Clean up old Docker images: `docker system prune -a`
2. Check backup folder size
3. Monitor with Coolify's resource monitoring

### SSL not working

1. Verify domain is pointing to your Coolify server's IP
2. Check Traefik logs in Coolify
3. Ensure port 80/443 is open

---

## Pre-Deploy Checklist

- [ ] Dockerfile created and builds successfully
- [ ] Image pushed to private registry
- [ ] Volume mounts configured in Coolify
- [ ] Environment variables set
- [ ] Domain configured with SSL
- [ ] Migrations configured (pre-deployment or entrypoint)
- [ ] Backup script created and tested
- [ ] Health check endpoint added

---

## Post-Deploy Checklist

- [ ] App loads without errors
- [ ] User can sign up / log in
- [ ] File uploads work (check storage volume)
- [ ] Stripe webhooks received (if applicable)
- [ ] Background jobs processing (if using Solid Queue)
- [ ] Backup runs successfully
- [ ] Logs show no errors
