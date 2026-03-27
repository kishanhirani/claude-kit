[Documentation Index](../index.md) / module2-api / AI Context

# AI Context: API

This file provides localised context for AI assistants working on the API documentation section.

---

## 🏗 Section Identity

The **API** section is the primary reference for developers integrating with Orderflow. It must be exact — method, path, auth requirement, request body, response body, and every possible error code. AI assistants editing this section must read the route files in `src/routes/` and the validator files in `src/validators/` before proposing any changes.

---

## 📚 Key Terminology

- **Resource:** A top-level API entity — `orders`, `payments`, `webhooks`
- **Status Transition:** A `PATCH /orders/:id/status` operation. Only valid transitions are accepted (see global AI context for the state machine)
- **Idempotency Key:** UUID v4 header required on all `POST /orders` and `POST /payments` calls
- **Webhook Signature:** HMAC-SHA256 header (`X-Webhook-Signature`) sent by fulfilment partners. Verified by `src/middleware/verifyWebhook.ts`
- **Stripe Webhook:** `POST /v1/webhooks/stripe` — verified using `stripe.webhooks.constructEvent()` with `STRIPE_WEBHOOK_SECRET`

---

## 💾 Core File Affiliation

- `src/routes/orders.ts` — orders router
- `src/routes/payments.ts` — payments router
- `src/routes/webhooks.ts` — webhooks router
- `src/validators/orderValidators.ts` — Zod schemas for order request bodies
- `src/validators/paymentValidators.ts` — Zod schemas for payment request bodies
- `src/middleware/requireAuth.ts` — JWT auth guard applied to all protected routes
- `src/middleware/idempotency.ts` — idempotency key middleware

---

## 🖇 Mandatory Cross-References

- `docs/module3-services/orderService.md` — business logic behind orders endpoints
- `docs/module3-services/paymentService.md` — business logic behind payments endpoints
- `docs/module4-dataModels/orderSchema.md` — Order schema fields and constraints
- `docs/module4-dataModels/paymentSchema.md` — Payment schema fields and constraints

---

## 🎨 Formatting Rules

- Every endpoint section must follow this order: method + path, description, auth requirement, request body, response body, error codes
- Use `json` code blocks for all request and response examples
- Use `typescript` code blocks only for type definitions if included
- Error codes must be listed in a table with columns: `HTTP Status`, `Code`, `Trigger Condition`
- Use `> [!WARNING]` for any security-sensitive behaviour (webhook verification, idempotency misuse)

---

## 🚨 Strict Constraints

1. Never document an endpoint that does not exist in `src/routes/` — read the file first
2. Never omit error codes — every documented endpoint must list all possible error responses
3. Never show real tokens, order IDs, or payment IDs in examples — use `[ORDER_ID]`, `[PAYMENT_ID]`, `[JWT_TOKEN]`
4. Webhook endpoint docs must always state that signature verification is mandatory and non-bypassable
