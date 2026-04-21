---
name: solid-suite-config
description: Use when working with background jobs, caching, or real-time websockets in Rails 8. Applies to Solid Queue (job processing), Solid Cache (fragment + Russian doll caching), Solid Cable (ActionCable adapter). Triggers when user mentions background job, perform_later, ActiveJob, rate limit, cache, fragment cache, ActionCable, Turbo broadcast, websocket, Redis alternative, Sidekiq alternative. Redis-free — database-backed, zero ops.
---

# Solid Suite Configuration — Queue + Cache + Cable

## Philosophy

> Database for everything. No Redis. No Sidekiq. Zero ops.

Rails 8 Solid Suite replaces traditional setup (Redis + Sidekiq + Memcached) with database-backed solutions:

- **Solid Queue** → job processing
- **Solid Cache** → caching layer
- **Solid Cable** → ActionCable adapter

## Solid Queue (Background Jobs)

```ruby
class DeliveryJob < ApplicationJob
  queue_as :default

  def perform(order)
    # async work
  end
end

DeliveryJob.perform_later(order)
```

Config in `config/queue.yml` — dispatchers, workers, threads.

## Solid Cache (Fragment Caching)

Russian doll pattern:
```erb
<% cache post do %>
  <h1><%= post.title %></h1>
  <% cache ["comments", post] do %>
    <%= render post.comments %>
  <% end %>
<% end %>
```

Config in `config/cache.yml` — max_size, expiry.

## Solid Cable (Broadcasts)

Model:
```ruby
class Item < ApplicationRecord
  belongs_to :list
  broadcasts_to :list
end
```

View:
```erb
<%= turbo_stream_from @list %>
```

Config in `config/cable.yml` — adapter, database connection.

## Anti-Patterns

- ❌ Suggesting Redis for caching → use Solid Cache
- ❌ Suggesting Sidekiq/Resque → use Solid Queue
- ❌ Suggesting Redis pub/sub → use Solid Cable
- ❌ Separate Redis server → database handles all

## References

- `references/background-jobs.md` — Solid Queue patterns, tenant preservation, continuable jobs
- `references/caching.md` — HTTP caching, fragment caching, invalidation
- `references/actioncable.md` — Multi-tenant WebSockets, broadcast scoping
