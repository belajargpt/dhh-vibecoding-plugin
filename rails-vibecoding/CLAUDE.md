# Rails Vibecoding — Project Conventions

Plugin ini ngeprovide Claude Code dengan expertise di stack BelajarGPT Vibecoding. Konteks berikut di-load di setiap session untuk bikin code generation + debugging + deployment konsisten dengan course convention.

## Stack (non-negotiable)

| Layer | Choice | Rationale |
|---|---|---|
| Framework | **Rails 8.1** | Majestic monolith, AI-friendly, opinionated |
| Frontend | **Hotwire (Turbo + Stimulus)** + importmap | No JS toolchain, AI handles HTML+ERB end-to-end |
| Database | **SQLite + Litestream** | Zero DB ops, file-based, backup to cloud |
| Queue / Cache / Cable | **Solid Queue / Solid Cache / Solid Cable** | No Redis, database-backed |
| Deployment | **Kamal 2** → VPS (Hetzner / Biznet) | Own infrastructure, Docker-based |
| Payment | **Mayar** (`api.mayar.id`) | Indonesian payment gateway, hosted checkout |
| Cloud storage | **Local filesystem** (v1) / R2 optional upgrade | Simplicity for v1, upgrade path documented |
| LLM integration | **RubyLLM gem** | Provider-agnostic (Claude/OpenAI/Gemini), streaming, cost-aware |
| Primary AI tool | **Claude Code** | Student's coding partner throughout |

## Vibecoding Principles (Non-Negotiable)

1. **Student is non-programmer.** Pri: PRD first (planning), then prompt. Explain via analogies, not jargon. When student asks "bikin X", assume minimal context — guide step by step.

2. **Prefer Rails built-in over adding gems.** Low-dependency rule. If Rails core / Solid Suite can do it, don't suggest a gem.

3. **Opinionated one-way implementations.** Don't offer 3 alternatives — pick the DHH-style default and implement it. Student isn't equipped to choose between patterns.

4. **Commit + push via prompt, not custom commands.** Student prompts *"commit dengan message jelas + push ke GitHub"* — you handle git via Bash. No custom slash commands exist in this plugin.

5. **Use Kamal directly for deploy.** `kamal setup`, `kamal deploy`, `kamal rollback`. Don't wrap these in custom commands.

6. **Mobile-first.** 70%+ Indonesian users on mobile. All UI generated must be mobile-responsive by default. Test in mobile viewport before declaring done.

7. **No SMTP / email dependency.** Course deliberately skips SMTP. For password reset: admin resets manually from admin panel. For digital product delivery: on-screen signed URL with 24hr expiry (no email delivery).

8. **Active Storage uses local filesystem.** Don't configure R2/S3 — local filesystem on VPS disk is sufficient for v1. Mention R2 as optional upgrade in comments.

## Student Context

- **Primary audience:** Mid-career Indonesian professional 35+ (non-programmer)
- **Background:** Marketer, founder, creator, consultant, UKM owner — has career + family, limited time, may have failed at "learn to code" before
- **Goal:** Ship real web apps (not become senior dev)
- **Language:** Bahasa Indonesia primary, English technical terms preserved
- **Environment:** Mac-first, Windows via WSL2

## Common Workflows

### Save point habit (every session)
Student prompts: *"commit perubahan dengan message jelas + push ke GitHub"*
→ You: read diff, generate descriptive commit message, `git add` + `git commit -m "..."` + `git push origin main`

### Deploy (M3+)
Student prompts: *"cek app siap deploy + deploy"*
→ You: run `bin/rails test`, validate Kamal config, validate SSH + DNS + Docker, THEN execute `kamal deploy`. Use `kamal rollback` if deploy breaks.

### Error debugging
Student pastes error verbatim + context.
→ You: read error, check `log/development.log` or `kamal app logs`, diagnose, propose fix. If unsure, ask clarifying question rather than guess.

### Multi-app VPS
Course pattern (M4+): student has 1 VPS hosting 4-5 apps via Kamal proxy + subdomains.
→ When deploying Nth app, update `deploy.yml` with subdomain host, add DNS A record instruction, keep previous apps running.

## What NOT to Do

- ❌ Don't suggest devise (use Rails 8 built-in auth)
- ❌ Don't suggest pundit / cancancan (simple role checks in models / controllers)
- ❌ Don't suggest sidekiq (Solid Queue)
- ❌ Don't suggest redis (database for everything)
- ❌ Don't suggest view_component (partials work)
- ❌ Don't suggest GraphQL (REST with Turbo sufficient)
- ❌ Don't suggest factory_bot (fixtures are simpler)
- ❌ Don't suggest rspec (Minitest ships with Rails)
- ❌ Don't suggest Tailwind toolchain setup (use Tailwind via CDN for v1, or simple tailwindcss-rails gem)
- ❌ Don't suggest CI/CD setup beyond Kamal
- ❌ Don't over-explain programming concepts — student is product owner, not programmer

## Reference Source

DHH-style Rails conventions di-inform oleh [Unofficial 37signals Coding Style Guide](https://github.com/marckohlbrugge/unofficial-37signals-coding-style-guide) (transferable patterns dari 37signals Fizzy codebase).
