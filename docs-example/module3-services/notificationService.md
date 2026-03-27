[Documentation Index](../index.md) / module3-services / Notification Service

# Notification Service

**File:** `src/services/NotificationService.ts`

The Notification Service is the single outbound communication layer in Orderflow. It receives typed events from `OrderService` and `PaymentService` and dispatches emails via SMTP (Nodemailer).

---

## Design Principles

- **Fire-and-forget:** `emit()` never throws. Callers do not await a success/failure result.
- **Retry:** Each notification is retried up to 3 times with exponential backoff (1s, 2s, 4s) before being logged as permanently failed.
- **No coupling:** Services emit events — they do not construct email content. Template selection and recipient lookup happen inside `NotificationService`.

---

## Methods

### `emit(event: NotificationEvent, payload: Record<string, unknown>): void`

Emits a typed notification event. Returns immediately. Processing is asynchronous.

```typescript
// Example usage inside OrderService
this.notificationService.emit('order.created', { orderId: order.id, customerId: order.customerId });
```

---

## Supported Event Types

| Event | Recipient | Template |
|---|---|---|
| `order.created` | Customer | `order-confirmation.html` |
| `order.cancelled` | Customer | `order-cancelled.html` |
| `order.fulfilled` | Customer | `order-dispatched.html` |
| `order.delivered` | Customer | `order-delivered.html` |
| `payment.initiated` | Customer | `payment-pending.html` |
| `payment.failed` | Customer | `payment-failed.html` |
| `fulfilment.routing_failed` | Ops team | `ops-routing-failure.html` |

---

## Failure Behaviour

If all 3 retry attempts fail, the service:

1. Logs the failure at `error` level: `logger.error({ event, payload, attempts: 3 }, 'Notification permanently failed')`
2. Does **not** throw
3. Does **not** persist a failed notification record (no dead-letter queue — this is a known limitation tracked in the roadmap)

> [!WARNING]
> There is currently no visibility into permanently failed notifications beyond log aggregation. If notification delivery guarantees are required for a new event type, discuss with the team before using `emit()` — a queue-backed approach may be needed.

---

## SMTP Configuration

Configured via environment variables:

| Variable | Purpose |
|---|---|
| `SMTP_HOST` | SMTP server hostname |
| `SMTP_PORT` | SMTP server port (typically 587 for STARTTLS) |
| `SMTP_USER` | SMTP authentication username |
| `SMTP_PASS` | SMTP authentication password |
| `NOTIFICATION_FROM` | Sender address on all outbound emails |
