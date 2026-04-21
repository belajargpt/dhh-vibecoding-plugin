---
name: mayar-payment-integration
description: Use when integrating Mayar payment gateway in Rails apps. Applies to hosted checkout integration, webhook handling, payment verification, digital product delivery with signed URLs, and Indonesian payment gateway patterns. Triggers when user mentions Mayar, pembayaran, checkout, QRIS, webhook, payment gateway, bayar, Indonesian payment, or hosted checkout. Also applies to similar gateways (Tripay, Xendit, iPaymu, Duitku) — pattern transfers.
---

# Mayar Payment Integration

## Philosophy

> Hosted checkout. Webhook + re-fetch. Never trust webhook body alone.

Mayar = Indonesian payment gateway with hosted checkout pattern:
1. Buyer clicks "Beli" on app
2. App creates payment request → redirects buyer to Mayar
3. Buyer pays at Mayar (QRIS, transfer, card)
4. Mayar redirects back to app + POSTs webhook
5. App verifies payment via re-fetch, fulfills

## API Endpoints

| Environment | URL |
|---|---|
| Sandbox | `https://api.mayar.club` |
| Production | `https://api.mayar.id` |

Store in `Rails.application.credentials.mayar.base_url` + `api_key`.

## Payment Creation Pattern

```ruby
class MayarPaymentService
  def self.create(order:, amount:, description:)
    response = Faraday.post("#{base_url}/hl/v1/payment/create") do |req|
      req.headers["Authorization"] = "Bearer #{api_key}"
      req.headers["Content-Type"] = "application/json"
      req.body = {
        name: order.buyer_name,
        email: order.buyer_email,
        amount: amount,
        description: description,
        webhookUrl: "#{Rails.application.routes.url_helpers.root_url}webhooks/mayar/#{webhook_token}"
      }.to_json
    end
    JSON.parse(response.body)
  end
end
```

Returns `data.link` → redirect buyer to that URL.

## ⚠️ Critical Security Pattern — Webhook Re-fetch

**Mayar webhooks have NO HMAC signature.** Attacker can POST fake "payment confirmed" to webhook URL. Kalau app trusts webhook body, attacker gets product free.

### Defense: re-fetch pattern

```ruby
class WebhooksController < ApplicationController
  skip_before_action :verify_authenticity_token  # webhooks are POST without CSRF

  def mayar
    return head :not_found unless params[:token] == Rails.application.credentials.mayar.webhook_token

    order_id = params[:order_id]

    # ⚠️ Don't trust params[:status]. Re-fetch from Mayar API.
    mayar_payment = MayarPaymentService.fetch(order_id: order_id)

    if mayar_payment["status"] == "paid"
      Order.find(order_id).mark_as_paid!
    end

    head :ok
  end
end
```

**Rule:** Always re-fetch from source of truth before fulfilling. Same pattern for Tripay, Xendit, iPaymu.

## Hard-to-Guess Webhook URL

Use random token in URL path:
```ruby
# config/routes.rb
post "/webhooks/mayar/:token", to: "webhooks#mayar"
```

Token stored in credentials. Not the only defense (re-fetch is primary) but adds obscurity layer.

## Digital Product Delivery (No Email)

After payment confirmed, redirect buyer to signed download URL:

```ruby
class OrdersController < ApplicationController
  def show
    @order = Order.find_signed!(params[:signed_token], purpose: :download, expires_in: 24.hours)
  end
end

# Generate signed URL after payment
order.signed_id(purpose: :download, expires_in: 24.hours)
```

No SMTP needed. Buyer bookmarks page. QR code for save-to-phone.

## State Machine (Order lifecycle)

```ruby
class Order < ApplicationRecord
  enum status: { pending: 0, paid: 1, refunded: 2, expired: 3 }

  def mark_as_paid!
    return if paid?  # idempotent — webhook retries OK
    update!(status: :paid, paid_at: Time.current)
    DeliveryJob.perform_later(self)
  end
end
```

Idempotency critical — webhooks can fire multiple times.

## References

- `references/webhooks.md` — SSRF protection, delinquency tracking, webhook state machines, retry patterns
