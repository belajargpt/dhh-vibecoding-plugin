---
name: mayar-payment-integration
description: Use when integrating Mayar payment gateway in Rails apps. Applies to hosted checkout integration, payment creation, webhook registration + handling, payment status verification, and Indonesian payment gateway patterns. Triggers when user mentions Mayar, pembayaran, checkout, QRIS, webhook, payment gateway, bayar, Indonesian payment, or hosted checkout. Pattern also transfers to similar gateways (Tripay, Xendit, iPaymu, Duitku) with endpoint substitution.
---

# Mayar Payment Integration

> Verified against Mayar docs at https://docs.mayar.id/ — Headless API v1.

## Philosophy

> Hosted checkout. Register webhook separately. Always re-fetch payment status before fulfilling.

## Base URLs

| Environment | URL |
|---|---|
| Production | `https://api.mayar.id/hl/v1` |
| Sandbox | `https://api.mayar.club/hl/v1` |

Store `base_url` + `api_key` in Rails credentials:
```yaml
mayar:
  base_url: https://api.mayar.club/hl/v1
  api_key: mayar-xxx...
  webhook_token: random-string-in-url-path
```

## Authentication

All requests use Bearer token:
```
Authorization: Bearer {API_KEY}
```

## Create Payment Request

**Endpoint:** `POST /hl/v1/payment/create`

**Required body:**
```json
{
  "name": "string",
  "email": "string",
  "amount": 170000,
  "mobile": "08996136751",
  "redirectUrl": "https://yourapp.com/orders/{token}/thank-you",
  "description": "Hook Content Playbook",
  "expiredAt": "2025-12-29T09:41:09.401Z"
}
```

**Response:**
```json
{
  "statusCode": 200,
  "messages": "success",
  "data": {
    "id": "uuid",
    "transaction_id": "uuid",
    "link": "https://mayar.link/..."
  }
}
```

Redirect buyer to `data.link`. After payment, Mayar redirects back to `redirectUrl`.

### Rails Service Class
```ruby
class MayarPaymentService
  def self.create(order:, amount:, description:, redirect_url:)
    conn = Faraday.new(url: credentials[:base_url]) do |f|
      f.request :json
      f.response :json
    end

    response = conn.post("payment/create") do |req|
      req.headers["Authorization"] = "Bearer #{credentials[:api_key]}"
      req.body = {
        name: order.buyer_name,
        email: order.buyer_email,
        amount: amount,
        mobile: order.buyer_mobile,
        redirectUrl: redirect_url,
        description: description,
        expiredAt: 24.hours.from_now.iso8601(3)
      }
    end

    response.body.dig("data", "link")
  end

  def self.credentials
    Rails.application.credentials.mayar
  end
end
```

## Register Webhook URL (One-Time Setup)

**Endpoint:** `GET /hl/v1/webhook/register` (yes, GET with body per docs)

**Body:**
```json
{ "urlHook": "https://yourapp.com/webhooks/mayar/{random-token}" }
```

Done once per merchant account (via script or dashboard). URL must be publicly accessible.

## Webhook Events

Mayar sends POST to your `urlHook` with JSON body. Event types:

- `payment.received` — payment completed
- `payment.reminder` — after 29 min, unpaid
- `shipper.status` — physical shipping
- `membership.*` — membership events (5 sub-types)

### Webhook payload (key fields)
```json
{
  "event": "payment.received",
  "data": {
    "id": "uuid",
    "status": "paid",
    "createdAt": "...",
    "updatedAt": "..."
  },
  "merchantId": "...",
  "customerName": "...",
  "customerEmail": "...",
  "amount": 170000,
  "productName": "..."
}
```

## ⚠️ CRITICAL — Re-fetch Before Fulfilling

**Mayar webhooks have NO HMAC signature.** Attacker can POST fake `payment.received` to your webhook URL → if app trusts body, attacker gets product free.

**Defense: always re-fetch from source of truth.**

### Re-fetch Endpoint
`GET /hl/v1/payment/{id}` with Bearer auth. Returns current status.

### Webhook Controller Pattern
```ruby
class WebhooksController < ApplicationController
  skip_before_action :verify_authenticity_token  # webhooks bypass CSRF

  def mayar
    return head :not_found unless params[:token] == Rails.application.credentials.mayar[:webhook_token]

    event = params[:event]
    payment_id = params.dig(:data, :id)

    case event
    when "payment.received"
      # ⚠️ Don't trust params[:data][:status]. Re-fetch.
      status = MayarPaymentService.fetch_status(payment_id)
      Order.find_by!(mayar_payment_id: payment_id).mark_as_paid! if status == "paid"
    end

    head :ok
  end
end
```

```ruby
class MayarPaymentService
  def self.fetch_status(payment_id)
    conn = Faraday.new(url: credentials[:base_url])
    response = conn.get("payment/#{payment_id}") do |req|
      req.headers["Authorization"] = "Bearer #{credentials[:api_key]}"
    end
    JSON.parse(response.body).dig("data", "status")
  end
end
```

## Hard-to-Guess Webhook URL (Secondary Defense)

Use random token in URL path (already shown above):
```ruby
# config/routes.rb
post "/webhooks/mayar/:token", to: "webhooks#mayar"
```

Obscurity adds layer but re-fetch is primary defense.

## Order State Machine (Idempotent)

Webhooks can retry — handler must be idempotent:
```ruby
class Order < ApplicationRecord
  enum status: { pending: 0, paid: 1, refunded: 2, expired: 3 }

  def mark_as_paid!
    return if paid?  # idempotent — safe on webhook retry
    update!(status: :paid, paid_at: Time.current)
    DeliveryJob.perform_later(self)
  end
end
```

## Scope Boundaries (Student Responsibility)

- NPWP/KTP untuk merchant active — student's responsibility, not covered in integration.
- Refund flows — handle via Mayar dashboard manually for v1.
- Chargebacks — handle manually, not in scope.

## References

- `references/webhooks.md` — general webhook patterns (SSRF protection, delinquency tracking, state machines) from 37signals

## Authoritative Source

Official docs: https://docs.mayar.id/ — check for API changes, new endpoints, deprecations before relying on this skill.
