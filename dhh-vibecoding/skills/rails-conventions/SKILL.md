---
name: rails-conventions
description: Use when writing Ruby on Rails code in DHH's 37signals style. Applies to creating models, controllers, views, routes, concerns, or any Rails application code. Triggers on Rails scaffolding, CRUD generation, refactoring, code review, or when the user mentions DHH, 37signals, Basecamp, HEY, Fizzy, vanilla Rails, fat model / skinny controller, REST, resource routing, or low-dependency style. Embodies REST purity, rich domain models, concerns over service objects, database-backed state, and the "prefer Rails built-in over adding gems" principle.
---

# Rails Conventions — DHH / 37signals Style

## Core Philosophy

> Vanilla Rails is plenty. The best code is the code you don't write.

- **Rich domain models** over service objects
- **CRUD controllers** over custom actions
- **Concerns** for horizontal code sharing
- **Records as state** instead of boolean columns
- **Database-backed everything**
- **Build solutions before reaching for gems**

## Golden Rules

1. **Fat models, skinny controllers.** Business logic in models.
2. **REST purity.** Resource-based routes. Custom actions = reconsider as new resource.
3. **Concerns > service objects.** Shared behavior = extract to concern.
4. **Prefer Rails built-in.** Don't add gems for what Rails already does.
5. **Convention over configuration.** Follow Rails defaults religiously.

## What to Avoid

- ❌ devise (Rails 8 built-in auth)
- ❌ pundit / cancancan (simple role checks)
- ❌ sidekiq (Solid Queue)
- ❌ redis (database-backed everything)
- ❌ view_component (partials work)
- ❌ GraphQL (REST with Turbo)
- ❌ factory_bot (fixtures)
- ❌ rspec (Minitest)

## References

Deep dive in `references/`:
- `controllers.md` — thin controllers, composable concerns, naming
- `models.md` — rich domain models, concerns, Current context, PORO
- `views.md` — Turbo Streams, partials, fragment caching, helpers
- `routing.md` — everything is CRUD, resource patterns
- `database.md` — UUIDs, state as records, database-backed everything
- `development-philosophy.md` — Ship/Validate/Refine, DHH review patterns
- `what-they-avoid.md` — explicit list of patterns 37signals skips
- `dhh.md` — DHH's code review patterns + philosophy

Consult relevant reference when pattern-specific depth needed.
