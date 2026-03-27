[Documentation Index](../index.md) / module3-services / AI Context

# AI Context: Services

This file provides localised context for AI assistants working on the Services documentation section.

---

## 🏗 Section Identity

The **Services** section documents the business logic layer of Orderflow. Every service is a TypeScript class with injected dependencies. AI assistants editing this section must read the actual service source files before proposing any changes — never document behaviour inferred from route files alone.

---

## 📚 Key Terminology

- **AppError:** The typed error class at `src/errors/AppError.ts`. All services throw `AppError` instances — never plain `Error` objects or string throws.
- **State Machine:** The order lifecycle enforced in `OrderService.ts`. Illegal transitions throw `AppError('INVALID_STATE_TRANSITION', 400, ...)`.
- **Fire-and-forget:** `NotificationService.emit()` is non-blocking and non-throwing. It logs failures and retries internally. Callers must not await it if they want to avoid coupling to notification latency.
- **PaymentIntent:** A Stripe object created by `PaymentService.createIntent()`. It is persisted as a `Payment` document in MongoDB immediately on creation, before Stripe confirms.
- **Idempotency Guard:** Applied at the middleware layer before any service method is called. Services do not re-implement idempotency — they assume the middleware has already handled it.

---

## 💾 Core File Affiliation

- `src/services/OrderService.ts` — order creation, state transitions, fulfilment routing
- `src/services/PaymentService.ts` — Stripe PaymentIntent creation and status sync
- `src/services/NotificationService.ts` — event emission, SMTP dispatch, retry logic
- `src/errors/AppError.ts` — typed error class
- `src/errors/errorCodes.ts` — all valid error code constants

---

## 🖇 Mandatory Cross-References

- `docs/module2-api/ordersEndpoints.md` — HTTP surface that delegates to OrderService
- `docs/module4-dataModels/orderSchema.md` — Order schema that OrderService reads and writes
- `docs/module4-dataModels/paymentSchema.md` — Payment schema that PaymentService reads and writes

---

## 🎨 Formatting Rules

- Use `typescript` code blocks for all method signatures and error throws
- Use Mermaid `flowchart` for service call chains
- Use `> [!NOTE]` for fire-and-forget behaviour — it is a common source of confusion
- Use `> [!WARNING]` for any behaviour that could cause data inconsistency if misunderstood

---

## 🚨 Strict Constraints

1. Never document a service method that does not exist in the source file — read it first
2. Never suggest that `NotificationService.emit()` can be awaited for a success/failure result — it is intentionally fire-and-forget
3. Never suggest adding a try/catch around service calls in controllers — the global error handler in `src/middleware/errorHandler.ts` catches all `AppError` instances
4. Integration tests only — never document or suggest mocking the database layer in tests
