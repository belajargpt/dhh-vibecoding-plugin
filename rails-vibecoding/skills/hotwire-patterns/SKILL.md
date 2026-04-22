---
name: hotwire-patterns
description: Use when building reactive UI in Rails apps without a JavaScript framework. Applies to Turbo Frames, Turbo Streams, Stimulus controllers, real-time updates, inline editing, live search, drag-and-drop, morphing, broadcasts, and any "no full page reload" UX. Triggers when user mentions Hotwire, Turbo, Stimulus, real-time, broadcasting, reactive UI, live update, inline edit, drag-drop, or websockets.
---

# Hotwire Patterns — Turbo + Stimulus

## Philosophy

> HTML as the wire protocol. Modern UX without JS framework.

- **Turbo Frames** = isolated sections that update independently
- **Turbo Streams** = server pushes updates to browsers (append / prepend / replace / remove)
- **Stimulus** = sprinkle behavior onto HTML (not a framework)

## Core Patterns

### Turbo Frame (inline update)
```erb
<turbo-frame id="post_<%= post.id %>">
  <%= post.title %>
  <%= link_to "Edit", edit_post_path(post) %>
</turbo-frame>
```
Click "Edit" → response returns turbo-frame with form → browser swaps just that frame.

### Stimulus controller
```html
<div data-controller="dropdown" data-action="click@window->dropdown#close">
  <button data-action="click->dropdown#toggle">Menu</button>
</div>
```

## Real-Time Broadcasts — 2 Patterns

Both are valid Rails 8. Pick based on complexity.

### Pattern A — Declarative macro (simpler)

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

Auto-broadcasts `create` / `update` / `destroy` to subscribers. Uses default partial convention (`_item.html.erb`).

**Use when:** standard CRUD broadcasts, default partial naming works, no conditional logic.

### Pattern B — Manual `broadcast_*_later_to` (flexible)

From 37signals Fizzy codebase:
```ruby
class Pin < ApplicationRecord
  after_update_commit :broadcast_pin_updates, if: :preview_changed?

  private

  def broadcast_pin_updates
    broadcast_replace_later_to [ user, :pins_tray ],
      partial: "my/pins/pin",
      target: "pin_#{id}"
  end
end
```

View:
```erb
<%= turbo_stream_from [current_user, :pins_tray] %>
```

Methods available:
- `broadcast_append_later_to`
- `broadcast_prepend_later_to`
- `broadcast_replace_later_to`
- `broadcast_remove_to`
- `broadcast_update_later_to`

**Use when:**
- Conditional broadcasts (`if: :some_condition_changed?`)
- Custom partial or target
- Composite scope `[parent, :namespace]`
- Want to broadcast from callback methods
- Need to broadcast to multiple targets

### Decision Guide

| Need | Pattern |
|---|---|
| Simple CRUD broadcasts | Pattern A (`broadcasts_to`) |
| Conditional on field change | Pattern B (`if: :field_changed?`) |
| Custom partial / target | Pattern B |
| Multiple broadcast targets | Pattern B |
| Debugging / explicit control | Pattern B |

## Other Common Needs

| Need | Use |
|---|---|
| Edit item inline | Turbo Frame |
| Delete item, list updates | Turbo Frame or Stream |
| Dropdown / modal toggle | Stimulus |
| Drag-drop reorder | Stimulus + Turbo Frame |
| Live search | Turbo Frame with debounced input |
| Streaming text (AI chatbot) | Turbo Stream append |

## References

- `references/hotwire.md` — Turbo Frames, Streams, morphing, drag & drop
- `references/stimulus.md` — reusable controller catalog with copy-paste code
