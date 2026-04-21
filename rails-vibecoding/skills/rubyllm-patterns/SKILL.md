---
name: rubyllm-patterns
description: Use when integrating LLM APIs (Claude, OpenAI, Gemini) in Rails apps via the RubyLLM gem. Applies to AI chatbots, streaming responses, auto-categorization / classification, structured outputs, cost-aware model selection, rate limiting, graceful degradation. Triggers when user mentions AI, LLM, Claude API, OpenAI, Gemini, chatbot, streaming response, auto-categorize, classification, RubyLLM, Anthropic, Net::HTTP AI, or any language model integration in Rails.
---

# RubyLLM Patterns

## Philosophy

> One interface, all providers. No Net::HTTP plumbing. Cost-aware by default.

RubyLLM gem abstracts LLM APIs behind a single interface:
- Claude (Anthropic)
- OpenAI (GPT-4, GPT-5, etc.)
- Gemini (Google)
- Swap provider = change one line

## Basic Usage

```ruby
# In Gemfile
gem "ruby_llm"
```

```ruby
# Simple ask
response = RubyLLM.chat
  .with_model("claude-opus-4-7")
  .ask("Halo, apa kabar?")

puts response.content
```

API key storage:
```yaml
# bin/rails credentials:edit
anthropic:
  api_key: sk-ant-xxx...
openai:
  api_key: sk-xxx...
```

Access: `Rails.application.credentials.dig(:anthropic, :api_key)` — gem handles automatically if following convention.

## Cost-Aware Model Selection

Pick cheapest model that solves task:

| Task | Model | Cost |
|---|---|---|
| Classification (auto-categorize) | `claude-haiku-4-5` | ~$1/1M input tokens |
| Chat with reasoning | `claude-sonnet-4-6` | ~$3/1M input tokens |
| Complex reasoning | `claude-opus-4-7` | ~$15/1M input tokens |

```ruby
# Cheap classification
category = RubyLLM.chat.with_model("claude-haiku-4-5")
  .ask("Category for: '#{description}' from: food, transport, entertainment")

# Reasoning chat
answer = RubyLLM.chat.with_model("claude-sonnet-4-6")
  .with_context(user_data)
  .ask(question)
```

## Streaming Response (via Turbo Stream)

Pattern: AI streams tokens → Turbo Stream broadcasts to view → user sees live typing.

```ruby
class ChatController < ApplicationController
  def create
    message = @chat.messages.create!(role: "user", content: params[:prompt])
    StreamAiResponseJob.perform_later(@chat, message)
    redirect_to chat_path(@chat)
  end
end

class StreamAiResponseJob < ApplicationJob
  def perform(chat, user_message)
    assistant_message = chat.messages.create!(role: "assistant", content: "")

    RubyLLM.chat
      .with_model("claude-sonnet-4-6")
      .with_context(chat.context)
      .stream(user_message.content) do |chunk|
        assistant_message.update!(content: assistant_message.content + chunk.content)
        Turbo::StreamsChannel.broadcast_append_to(
          chat, target: "message_#{assistant_message.id}",
          html: chunk.content
        )
      end
  end
end
```

## Rate Limiting (Cost Control)

Rails 8 built-in:
```ruby
class ChatController < ApplicationController
  rate_limit to: 50, within: 1.day, by: -> { current_user.id }
end
```

Reflect budget constraint (e.g., 50 calls/day × user = manageable monthly cost).

## Graceful Degradation

External API = not always available. Never block core functionality:

```ruby
class Transaction < ApplicationRecord
  def auto_categorize!
    return if category.present?

    category = RubyLLM.chat.with_model("claude-haiku-4-5")
      .ask("Category for: #{description}")
    update(category: category.content.strip)
  rescue RubyLLM::Error => e
    Rails.logger.warn "AI categorize failed: #{e.message}"
    # Fallback: user inputs category manually via form
  end
end
```

## Structured Output

For classification / data extraction:
```ruby
response = RubyLLM.chat.with_model("claude-haiku-4-5")
  .with_schema({
    type: "object",
    properties: {
      category: { type: "string", enum: ["food", "transport", "entertainment"] },
      confidence: { type: "number" }
    }
  })
  .ask("Classify: #{description}")

# response.content is parsed JSON matching schema
```

## Anti-Patterns

- ❌ `Net::HTTP.post` directly to API — use RubyLLM
- ❌ Hardcoding API key in code — use Rails credentials
- ❌ Always using Opus — use Haiku for simple tasks
- ❌ No rate limit — users can drain budget
- ❌ No fallback when API down — user sees broken app

## References

- `references/ai-llm.md` — Command pattern, cost tracking, tool patterns
