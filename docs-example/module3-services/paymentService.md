[Documentation Index](../index.md) / module3-services / Payment Service

# Payment Service

**File:** `src/services/PaymentService.ts`

The Payment Service owns the Stripe PaymentIntent lifecycle and keeps the `Payment` MongoDB document in sync with Stripe's state via webhook events.

---

## Methods

### `createIntent(data: CreatePaymentDto): Promise<Payment>`

Initiates payment for a confirmed order.

1. Validates the order is in `confirmed` status — throws `INVALID_ORDER_STATUS` otherwise
2. Checks no payment already exists for the order — throws `PAYMENT_ALREADY_EXISTS` otherwise
3. Creates a Stripe PaymentIntent via `stripe.paymentIntents.create()`
4. Persists a `Payment` document with `status: 'pending'` and the Stripe intent ID
5. Emits `payment.initiated` notification event (fire-and-forget)
6. Returns the `Payment` document

> [!NOTE]
> The Stripe PaymentIntent is created with `confirm: true` and the supplied `paymentMethodId`. If Stripe declines synchronously, `createIntent()` throws `AppError('PAYMENT_DECLINED', 402)`. Async declines arrive via the `payment_intent.payment_failed` webhook.

---

### `syncFromStripeEvent(event: Stripe.Event): Promise<void>`

Called by the Stripe webhook handler. Maps Stripe events to Payment document updates.

| Stripe Event | Payment Status | Side Effect |
|---|---|---|
| `payment_intent.succeeded` | `succeeded` | Calls `OrderService.transitionStatus(orderId, 'processing')` |
| `payment_intent.payment_failed` | `failed` | Emits `payment.failed` notification event |
| `charge.refunded` | `refunded` | Calls `OrderService.transitionStatus(orderId, 'refunded')` |

> [!WARNING]
> `syncFromStripeEvent()` must be idempotent — Stripe delivers webhooks at least once. The method checks the current `Payment.status` before applying a transition and no-ops if the transition is already applied.

---

### `getById(id: string, customerId: string): Promise<Payment>`

Fetches a payment by ID. Throws `FORBIDDEN` if the linked order's `customerId` does not match.

---

## Error Reference

| Code | HTTP | Trigger |
|---|---|---|
| `PAYMENT_NOT_FOUND` | 404 | No payment with given ID |
| `FORBIDDEN` | 403 | Payment linked to different customer |
| `INVALID_ORDER_STATUS` | 400 | Order not in `confirmed` status |
| `PAYMENT_ALREADY_EXISTS` | 409 | Payment already exists for this order |
| `PAYMENT_DECLINED` | 402 | Stripe synchronous decline |
