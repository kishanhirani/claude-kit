[Documentation Index](../index.md) / module2-api / Webhooks Endpoints

# Webhooks Endpoints

Orderflow receives inbound webhooks from two sources: **fulfilment partners** (status updates on shipments) and **Stripe** (payment lifecycle events). Both endpoints require signature verification — requests that fail verification are rejected with `401`.

---

## POST /v1/webhooks/fulfilment

Receives status updates from fulfilment partners.

**Auth:** Webhook signature (`X-Webhook-Signature` header)
**Verification:** HMAC-SHA256 of raw request body using the partner's shared secret

> [!WARNING]
> This endpoint uses `express.raw()` as its body parser — not `express.json()`. The raw body must be preserved for signature verification. Never add `express.json()` middleware to this route.

### Request Headers

```
X-Webhook-Signature: [HMAC_SHA256_HEX]
X-Partner-Id: [PARTNER_ID]
Content-Type: application/json
```

### Request Body

```json
{
  "event": "shipment.dispatched",
  "orderId": "[ORDER_ID]",
  "trackingNumber": "JD000123456789",
  "estimatedDelivery": "2026-01-18",
  "timestamp": "2026-01-15T14:00:00.000Z"
}
```

### Supported Event Types

| Event | Triggered When | Order Status Transition |
|---|---|---|
| `shipment.dispatched` | Partner has dispatched the order | `processing → fulfilled` |
| `shipment.delivered` | Carrier confirms delivery | `fulfilled → delivered` |
| `shipment.failed` | Delivery attempt failed | No transition — triggers notification |
| `shipment.returned` | Item returned to partner | Triggers manual review workflow |

### Response — 200 OK

```json
{ "received": true }
```

Always return `200` promptly. Processing happens asynchronously. Returning a non-`2xx` will cause the partner to retry.

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `401` | `INVALID_SIGNATURE` | HMAC verification failed |
| `400` | `UNKNOWN_PARTNER` | `X-Partner-Id` does not match a registered partner |
| `400` | `UNKNOWN_EVENT` | Event type not in the supported list |

---

## POST /v1/webhooks/stripe

Receives payment lifecycle events from Stripe.

**Auth:** Stripe webhook signature (`Stripe-Signature` header)
**Verification:** `stripe.webhooks.constructEvent()` using `STRIPE_WEBHOOK_SECRET`

> [!WARNING]
> This endpoint uses `express.raw()`. The raw body is required for Stripe signature verification. Do not parse it as JSON before passing to `stripe.webhooks.constructEvent()`.

### Handled Stripe Events

| Stripe Event | Action |
|---|---|
| `payment_intent.succeeded` | Updates Payment status to `succeeded`, transitions order to `processing` |
| `payment_intent.payment_failed` | Updates Payment status to `failed`, emits `payment.failed` notification event |
| `charge.refunded` | Updates Payment status to `refunded`, transitions order to `refunded` |

All other Stripe events are acknowledged with `200` and ignored.

### Response — 200 OK

```json
{ "received": true }
```

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `401` | `INVALID_STRIPE_SIGNATURE` | `stripe.webhooks.constructEvent()` threw |
| `400` | `UNHANDLED_EVENT` | Only returned if `STRICT_WEBHOOK_MODE=true` env var is set |
