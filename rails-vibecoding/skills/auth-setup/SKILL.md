---
name: auth-setup
description: Use when implementing authentication, authorization, or permission patterns in Rails 8. Applies to login, signup, session, password reset, current_user, role-based access, admin panel, multi-tenant scoping, permission isolation testing, Rails credentials. Triggers when user mentions auth, login, signup, session, current_user, password, admin, role, permission, isolation test, multi-tenant, membership, workspace, has_secure_password, credentials, or secret keys. Uses Rails 8 built-in auth — no devise, no pundit, no cancancan.
---

# Auth Setup — Rails 8 Built-in

## Philosophy

> Rails 8 ships a 150-line auth generator. It's enough.

- **Authentication** = "who are you?" → login, session
- **Authorization** = "what can you do?" → role check, ownership scoping

Two concepts, two layers.

## Authentication

Generate:
```bash
bin/rails generate authentication
```

Produces:
- `User` model (email, `has_secure_password`)
- `Session` model (multi-device, remember-me)
- `SessionsController` (login/logout)
- `RegistrationsController` (signup)
- `PasswordsController` (reset — remove if no SMTP)
- `current_user` helper
- `before_action :require_authentication`

## Authorization — Simple Role Checks

```ruby
class User < ApplicationRecord
  # t.boolean :admin, default: false
end

class AdminController < ApplicationController
  before_action :require_admin

  private

  def require_admin
    redirect_to root_path, alert: "Forbidden" unless current_user.admin?
  end
end
```

No pundit/cancancan needed. Simple methods suffice.

## Permission Isolation (Critical for Multi-User)

Every query scoped by `current_user`:

```ruby
# ❌ BAD — returns any workspace
Workspace.find(params[:id])

# ✅ GOOD — scoped to current_user's memberships
current_user.workspaces.find(params[:id])
```

### Isolation test (mandatory for multi-user)
```ruby
test "member of workspace A cannot access workspace B" do
  sign_in_as users(:alice)
  get list_path(lists(:workspace_b_list))
  assert_response :not_found
end
```

Manual test + automated test. Never trust scope enforcement without verification.

## Rails Credentials (Secrets)

```bash
EDITOR=nano bin/rails credentials:edit
```

- Encrypted: `config/credentials.yml.enc`
- Key: `config/master.key` (`.gitignore`d, never commit)
- Access: `Rails.application.credentials.some_key`

Use for: `secret_key_base`, invite tokens, API keys, webhook tokens.

## Admin Panel Pattern

Config-based (safer than "first user = admin"):
```yaml
# config/admins.yml
- admin@example.com
```

```ruby
def current_user_admin?
  admin_emails = YAML.load_file(Rails.root.join("config/admins.yml"))
  admin_emails.include?(current_user.email)
end
```

## References

- `references/authentication.md` — passwordless magic links pattern (37signals), session management
- `references/multi-tenancy.md` — path-based tenancy, middleware, ActiveJob extensions
- `references/security-checklist.md` — XSS, CSRF, SSRF, rate limiting, authorization
