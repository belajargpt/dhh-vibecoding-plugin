---
name: rails-debug-helper
description: Use when diagnosing Rails errors, reading logs, debugging deploy failures, running quality / security checks, or verifying pre-flight conditions before deploy. Applies to error diagnosis (stack traces, routing errors, migration issues, credential errors), log inspection (`log/development.log`, `kamal app logs`), test failures, deploy pre-flight checks, and CI pipeline (Rubocop, Brakeman, Bundler Audit). Triggers when user mentions error, stack trace, "stuck", log, deploy failed, migration issue, test failure, debug, Brakeman, Rubocop, security scan, bin/ci, or unexpected behavior.
---

# Rails Debug Helper

## Philosophy

> Errors are information. Read them literally. Diagnose via logs. Fix root causes, not symptoms.

## Debug Loop Pattern

**Step 1:** Copy error verbatim (don't paraphrase).

**Step 2:** Gather context — what was attempted, what happened, relevant log.

**Step 3:** Diagnose before fixing. Check logs:
- `log/development.log` (local)
- `kamal app logs` (production)
- `bin/rails test` output (tests)

**Step 4:** Fix root cause, not symptom. Example: missing migration → run `bin/rails db:migrate`, don't delete the code that expects the column.

## Common Error Patterns

| Error | Likely Cause | Fix |
|---|---|---|
| `ActiveRecord::RecordNotFound` | Scoped query returns nothing | Check `current_user` scope |
| `NoMethodError: undefined method 'x' for nil` | Missing preload or association | Add `.includes(:x)` or `joins` |
| `Routing Error` | Resource not defined in `config/routes.rb` | Add `resources :x` |
| `PG::UndefinedTable` (or SQLite equiv) | Migration not run | `bin/rails db:migrate` |
| `Mysql2::Error: Access denied` | Wrong credentials | Check `config/credentials.yml.enc` |
| `SSL certificate problem` (Kamal deploy) | DNS not propagated | Wait + `kamal deploy` again |
| `Docker build timeout` | Slow network or large image | Use stable WiFi, check Dockerfile layers |

## Pre-Flight Check Before Deploy

Before running `kamal deploy`:
1. ✅ `bin/rails test` — all green
2. ✅ `git status` — nothing uncommitted
3. ✅ `dig yourname.com` — DNS returns correct IP
4. ✅ `ssh deploy@your-vps-ip` — SSH works
5. ✅ `docker --version` (on VPS) — Docker installed
6. ✅ `kamal envify` — credentials synced

## Log Reading Patterns

Production logs via Kamal:
```bash
kamal app logs              # live stream
kamal app logs -n 100       # last 100 lines
kamal app logs --grep ERROR # filter
```

Local logs:
```bash
tail -f log/development.log
```

Screenshot log + paste to Claude Code for diagnosis.

## CI / Quality Gates

Full pre-commit / pre-deploy pipeline (pattern from 37signals Fizzy `bin/ci`):

```bash
bin/rubocop              # Code style
bundle exec bundler-audit # Gem CVE check
bin/importmap audit      # JS pin CVE check
bin/brakeman             # Security static analysis
bin/rails test           # Unit + integration tests
bin/rails test:system    # Capybara system tests
```

Or unified:
```bash
bin/ci  # if you ship a script chaining all of the above
```

### Brakeman — Security Static Analysis
Catches common Rails security issues before runtime:
```bash
bundle add brakeman --group=development,test
bin/brakeman            # full scan
bin/brakeman -A -w 2    # CI mode
```

Scans for: SQL injection, XSS, mass assignment, unsafe redirects, command injection, hardcoded secrets.

### Bundler Audit — Dependency CVE Check
```bash
bundle add bundler-audit --group=development,test
bundle exec bundler-audit check --update
```

Alerts when gem dependencies have known CVEs.

### Rubocop — Style Consistency
```bash
bundle add rubocop rubocop-rails --group=development,test
bin/rubocop             # check all
bin/rubocop -a          # auto-fix safe
```

Use `rubocop-rails-omakase` config (DHH-approved defaults).

## Test Patterns (Minitest + Rails)

```ruby
# test/controllers/posts_controller_test.rb
class PostsControllerTest < ActionDispatch::IntegrationTest
  setup do
    @user = users(:alice)  # Fixtures, not factories
    sign_in_as(@user)
  end

  test "shows only user's own posts" do
    get posts_url
    assert_response :success
    assert_select ".post", count: @user.posts.count
  end
end
```

System tests (Capybara + Selenium):
```bash
bin/rails test:system
PARALLEL_WORKERS=1 bin/rails test:system  # debug flaky tests
```

## Anti-Patterns

- ❌ Paraphrasing error instead of copying verbatim — details matter
- ❌ Fixing symptom without understanding cause
- ❌ Guessing vs checking logs
- ❌ Disabling tests that fail (fix the bug, not the test)
- ❌ Skipping Brakeman before deploy — production security debt
- ❌ Ignoring Bundler Audit alerts — known CVEs shipped

## References

- `references/testing.md` — Minitest, fixtures over factories, integration tests
- `references/observability.md` — Structured logging, Yabeda metrics
