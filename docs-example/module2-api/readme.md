[Documentation Index](../index.md) / module2-api

# API Overview

Orderflow exposes a REST API over HTTPS. All endpoints require JWT authentication unless marked public. All request and response bodies are JSON.

---

## Table of Contents

- [Base URL](#base-url)
- [Authentication](#authentication)
- [Idempotency](#idempotency)
- [Error Format](#error-format)
- [Rate Limiting](#rate-limiting)
- [Endpoints](#endpoints)

---

## Base URL

```
https://api.orderflow.internal/v1
```

Local development:

```
http://localhost:3000/v1
```

---

## Authentication

All protected endpoints require a Bearer token in the `Authorization` header:

```
Authorization: Bearer [JWT_TOKEN]
```

Tokens are issued by the auth service and expire after the duration set in `JWT_EXPIRES_IN`. Expired tokens return `401 Unauthorized` with code `TOKEN_EXPIRED`.

---

## Idempotency

All `POST` requests to `/orders` and `/payments` **must** include an `Idempotency-Key` header:

```
Idempotency-Key: [UUID_V4]
```

- Keys are scoped per authenticated user
- Duplicate keys within 24 hours return the original response without re-executing the operation
- Missing keys return `400 Bad Request` with code `MISSING_IDEMPOTENCY_KEY`

> [!WARNING]
> Never reuse an idempotency key across different request bodies. The server returns the original response regardless of whether the new request body differs.

---

## Error Format

All errors follow a consistent envelope:

```json
{
  "error": {
    "code": "INVALID_STATE_TRANSITION",
    "message": "Order cannot transition from fulfilled to pending",
    "field": null
  }
}
```

- **`code`** — machine-readable error code (use this in client logic, not `message`)
- **`message`** — human-readable description
- **`field`** — set to the offending field name for validation errors, `null` otherwise

---

## Rate Limiting

| Tier | Limit |
|---|---|
| Authenticated requests | 1000 req / 15 min per user |
| Webhook inbound | 500 req / min per partner IP |
| Health check | Unlimited |

Rate limit headers are returned on every response:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1705312200
```

---

## Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/health` | None | Health check |
| `POST` | `/v1/orders` | Required | Create a new order |
| `GET` | `/v1/orders/:id` | Required | Get order by ID |
| `PATCH` | `/v1/orders/:id/status` | Required | Transition order status |
| `GET` | `/v1/orders` | Required | List orders (paginated) |
| `POST` | `/v1/payments` | Required | Initiate payment for an order |
| `GET` | `/v1/payments/:id` | Required | Get payment by ID |
| `POST` | `/v1/webhooks/fulfilment` | Signature | Receive fulfilment partner webhook |
| `POST` | `/v1/webhooks/stripe` | Signature | Receive Stripe payment webhook |

---

## Detailed Endpoint Documentation

- [Orders Endpoints](./ordersEndpoints.md)
- [Payments Endpoints](./paymentsEndpoints.md)
- [Webhooks Endpoints](./webhooksEndpoints.md)
