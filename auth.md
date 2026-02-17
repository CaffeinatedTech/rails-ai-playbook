# Authentication - Rails 8 + OmniAuth

> Rails 8 authentication generator + OmniAuth for social login.

---

## Overview

Rails 8 includes a built-in authentication generator that provides:
- Session-based authentication (no JWT)
- `has_secure_password` for email/password login
- `Current.user` pattern for accessing authenticated user
- Session model for tracking active sessions

We extend this with OmniAuth for social login (Google, OpenAI, etc.).

---

## Step 1: Generate Rails 8 Auth

```bash
bin/rails generate authentication
```

This creates:
- `app/models/user.rb` with `has_secure_password`
- `app/models/session.rb` for session tracking
- `app/models/current.rb` for `Current.user`
- `app/controllers/concerns/authentication.rb`
- `app/controllers/sessions_controller.rb`
- `app/controllers/passwords_controller.rb`
- Database migrations for users and sessions

---

## Step 2: Extend User Model for OAuth

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password validations: false  # Optional password for OAuth users
  has_many :sessions, dependent: :destroy

  # OAuth UIDs
  attribute :google_uid, :string
  attribute :openai_uid, :string

  # Validations
  validates :email_address, presence: true, uniqueness: true
  validates :password, length: { minimum: 8 }, allow_nil: true

  normalizes :email_address, with: ->(e) { e.strip.downcase }

  def full_name
    [first_name, last_name].compact.join(" ").presence || email_address.split("@").first
  end

  def initials
    full_name.split.map(&:first).join.upcase[0, 2]
  end

  # Find or create from OAuth
  def self.find_or_create_from_oauth(auth)
    provider = auth.provider
    uid = auth.uid

    # Find by provider UID first
    user = case provider
    when 'google_oauth2'
      find_by(google_uid: uid)
    when 'openai'
      find_by(openai_uid: uid)
    end

    # Then by email
    user ||= find_by(email_address: auth.info.email)

    if user
      # Link OAuth account if not already linked
      uid_column = "#{provider == 'google_oauth2' ? 'google' : provider}_uid"
      user.update!(uid_column => uid) if user.send(uid_column).blank?
    else
      # Create new user (no password required for OAuth)
      user = create!(
        email_address: auth.info.email,
        first_name: auth.info.name&.split&.first,
        last_name: auth.info.name&.split&.drop(1)&.join(' '),
        "#{provider == 'google_oauth2' ? 'google' : provider}_uid" => uid,
        email_verified_at: Time.current  # OAuth emails are pre-verified
      )
    end

    user
  end
end
```

---

## Step 3: Add OmniAuth

```ruby
# Gemfile
gem 'omniauth'
gem 'omniauth-google-oauth2'
gem 'omniauth-rails_csrf_protection'

# For OpenAI OAuth (custom strategy needed)
gem 'omniauth-oauth2'
```

```ruby
# config/initializers/omniauth.rb
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :google_oauth2,
    ENV['GOOGLE_CLIENT_ID'],
    ENV['GOOGLE_CLIENT_SECRET'],
    {
      scope: 'email,profile',
      prompt: 'select_account'
    }

  # Add OpenAI if needed (requires custom strategy)
  # provider :openai, ENV['OPENAI_CLIENT_ID'], ENV['OPENAI_CLIENT_SECRET']
end

OmniAuth.config.allowed_request_methods = [:post]
```

---

## Step 4: OAuth Controller

```ruby
# app/controllers/oauth_controller.rb
class OauthController < ApplicationController
  allow_unauthenticated_access

  def callback
    auth = request.env['omniauth.auth']
    user = User.find_or_create_from_oauth(auth)

    start_new_session_for(user)
    redirect_to after_login_path, notice: "Welcome, #{user.first_name}!"
  end

  def failure
    redirect_to login_path, alert: "Authentication failed. Please try again."
  end

  private

  def after_login_path
    stored_location || app_path
  end

  def stored_location
    session.delete(:return_to)
  end
end
```

```ruby
# config/routes.rb
# OAuth callbacks
get '/auth/:provider/callback', to: 'oauth#callback'
get '/auth/failure', to: 'oauth#failure'
```

---

## Step 5: Authentication Concern

The generated `Authentication` concern provides helper methods. Extend if needed:

```ruby
# app/controllers/concerns/authentication.rb
module Authentication
  extend ActiveSupport::Concern

  included do
    before_action :require_authentication
    helper_method :authenticated?, :current_user
  end

  class_methods do
    def allow_unauthenticated_access(**options)
      skip_before_action :require_authentication, **options
    end
  end

  private

  def authenticated?
    Current.session.present?
  end

  def current_user
    Current.user
  end

  def require_authentication
    resume_session || request_authentication
  end

  def resume_session
    if session_id = cookies.signed[:session_id]
      if session = Session.find_by(id: session_id)
        Current.session = session
        return true
      end
    end
    false
  end

  def request_authentication
    session[:return_to] = request.fullpath
    redirect_to login_path, alert: "Please log in to continue."
  end

  def start_new_session_for(user)
    session = user.sessions.create!(
      ip_address: request.remote_ip,
      user_agent: request.user_agent
    )
    Current.session = session
    cookies.signed.permanent[:session_id] = { value: session.id, httponly: true }
  end

  def terminate_session
    Current.session&.destroy
    cookies.delete(:session_id)
    Current.session = nil
  end
end
```

---

## Step 6: Current Model

```ruby
# app/models/current.rb
class Current < ActiveSupport::CurrentAttributes
  attribute :session
  delegate :user, to: :session, allow_nil: true
end
```

---

## Key Points

1. **No JWT** — Rails 8 uses cookie-based sessions
2. **Optional password** — OAuth users don't need a password
3. **`Current.user`** — Access authenticated user anywhere
4. **Session tracking** — Each login creates a Session record
5. **`allow_unauthenticated_access`** — Explicitly mark public actions

---

## Common Patterns

### Require auth except for certain actions
```ruby
class PagesController < ApplicationController
  allow_unauthenticated_access only: [:home, :pricing, :about]
end
```

### Store location for redirect after login
```ruby
def request_authentication
  session[:return_to] = request.fullpath
  redirect_to login_path
end

# After login:
redirect_to session.delete(:return_to) || app_path
```

### Check authentication in views
```jsx
const { auth } = usePage().props;

{auth.authenticated ? (
  <p>Welcome, {auth.user.first_name}!</p>
) : (
  <Link href={routes.login}>Log in</Link>
)}
```
