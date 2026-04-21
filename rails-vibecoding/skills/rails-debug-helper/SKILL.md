---
name: rails-debug-helper
description: Use when diagnosing Rails errors, reading logs, debugging deploy failures, or verifying pre-flight conditions before deploy. Applies to error diagnosis (stack traces, routing errors, migration issues, credential errors), log inspection (`log/development.log`, `kamal app logs`), test failures, deploy pre-flight checks (SSH, DNS, Docker, bin/rails test). Triggers when user mentions error, stack trace, "stuck", log, deploy failed, migration issue, test failure, debug, or any unexpected behavior.
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

## Anti-Patterns

- ❌ Paraphrasing error instead of copying verbatim — details matter
- ❌ Fixing symptom without understanding cause
- ❌ Guessing vs checking logs
- ❌ Disabling tests that fail (fix the bug, not the test)

## References

- `references/testing.md` — Minitest, fixtures over factories, integration tests
- `references/observability.md` — Structured logging, Yabeda metrics
