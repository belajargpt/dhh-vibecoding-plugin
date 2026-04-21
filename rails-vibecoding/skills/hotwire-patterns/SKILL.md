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

### Turbo Stream (real-time broadcast)
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
Changes propagate to all connected browsers.

### Stimulus controller
```html
<div data-controller="dropdown" data-action="click@window->dropdown#close">
  <button data-action="click->dropdown#toggle">Menu</button>
</div>
```

## When to Use What

| Need | Use |
|---|---|
| Edit item inline | Turbo Frame |
| Delete item, list updates | Turbo Frame or Stream |
| Real-time multi-user broadcast | Turbo Stream + `broadcasts_to` |
| Dropdown / modal toggle | Stimulus |
| Drag-drop reorder | Stimulus + Turbo Frame |
| Live search | Turbo Frame with debounced input |
| Streaming text (AI chatbot) | Turbo Stream append |

## References

- `references/hotwire.md` — Turbo Frames, Streams, morphing, drag & drop
- `references/stimulus.md` — reusable controller catalog with copy-paste code
