[Documentation Index](../index.md) / module2-api / Payments Endpoints

# Payments Endpoints

---

## POST /v1/payments

Initiate payment for a confirmed order. Creates a Stripe PaymentIntent and links it to the order.

**Auth:** Required
**Idempotency-Key:** Required

### Request Body

```json
{
  "orderId": "[ORDER_ID]",
  "paymentMethodId": "[STRIPE_PAYMENT_METHOD_ID]",
  "currency": "GBP"
}
```

- `orderId` must reference an order in `confirmed` status. Attempting payment on a `pending` or `cancelled` order returns `400`.
- `paymentMethodId` is a Stripe PaymentMethod ID (e.g. `pm_card_visa` in test mode)
- `currency` must match the order's currency

### Response — 201 Created

```json
{
  "id": "[PAYMENT_ID]",
  "orderId": "[ORDER_ID]",
  "stripePaymentIntentId": "[STRIPE_INTENT_ID]",
  "status": "pending",
  "amount": 5998,
  "currency": "GBP",
  "createdAt": "2026-01-15T09:05:00.000Z"
}
```

Payment `status` transitions asynchronously via Stripe webhooks. Poll `GET /v1/payments/:id` or listen to the `payment.succeeded` / `payment.failed` webhooks to track final state.

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `400` | `MISSING_IDEMPOTENCY_KEY` | `Idempotency-Key` header absent |
| `400` | `INVALID_ORDER_STATUS` | Order is not in `confirmed` status |
| `400` | `CURRENCY_MISMATCH` | Requested currency does not match order currency |
| `404` | `ORDER_NOT_FOUND` | No order with this ID |
| `402` | `PAYMENT_DECLINED` | Stripe declined the payment method |
| `409` | `PAYMENT_ALREADY_EXISTS` | A payment already exists for this order |

---

## GET /v1/payments/:id

Get a payment record by ID.

**Auth:** Required

### Response — 200 OK

```json
{
  "id": "[PAYMENT_ID]",
  "orderId": "[ORDER_ID]",
  "stripePaymentIntentId": "[STRIPE_INTENT_ID]",
  "status": "succeeded",
  "amount": 5998,
  "currency": "GBP",
  "createdAt": "2026-01-15T09:05:00.000Z",
  "updatedAt": "2026-01-15T09:05:04.000Z"
}
```

### Payment Status Values

| Status | Description |
|---|---|
| `pending` | PaymentIntent created, awaiting Stripe confirmation |
| `succeeded` | Payment captured successfully |
| `failed` | Payment declined or errored |
| `refunded` | Full refund processed |

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `404` | `PAYMENT_NOT_FOUND` | No payment with this ID |
| `403` | `FORBIDDEN` | Payment belongs to a different customer |
