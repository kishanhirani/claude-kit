[Documentation Index](./index.md)

# Global AI Context: Orderflow

This file provides the primary context for any AI assistant interacting with the Orderflow documentation. It defines project identity, technical standards, conventions, and documentation maintenance rules.

---

## 🏗 Project Identity & Architecture

- **Project Name:** Orderflow
- **Core Purpose:** A Node.js/Express REST API for order lifecycle management. Handles order creation, payment processing, fulfilment routing, and customer notification across multiple fulfilment partners.
- **Platform:** Node.js 20 LTS, Express 4, MongoDB 7 (Mongoose 8)
- **Language:** TypeScript
- **Documentation Format:** Markdown (CommonMark), Mermaid.js for diagrams
- **Repository Type:** Backend API service — source lives in `src/`, tests in `tests/`, infrastructure in `infra/`

---

## 📚 Documentation Standards

### Mandatory Reading

Before proposing any documentation changes, the AI **must** review:

- `docs/module6-docMaintenance/documentationMaintenanceGuide.md` — workflow, sync rules, and formatting standards
- `docs/index.md` — current documentation structure and all active links

### Key Project Terminology

- **Order:** A customer purchase record. Has a lifecycle: `pending → confirmed → processing → fulfilled → delivered | cancelled | refunded`.
- **Fulfilment Partner:** An external logistics provider (e.g. WarehouseA, DropshipperB). Each partner has its own webhook contract.
- **Payment Intent:** A Stripe PaymentIntent created on order confirmation. Linked 1:1 to an order.
- **Notification Event:** A typed event emitted by the Notification Service to trigger emails/SMS. Events are fire-and-forget with retry logic.
- **Idempotency Key:** A client-supplied UUID required on all mutating requests to prevent duplicate order creation.
- **Webhook Signature:** An HMAC-SHA256 signature sent by fulfilment partners on inbound webhooks. Must be verified before processing.

---

## ✍️ Writing Styles & Rules

### Naming Conventions

- All folder names: **camelCase** (e.g., `module1-gettingStarted/`, `module3-services/`)
- All file names: **camelCase** (e.g., `orderSchema.md`, `paymentService.md`)
- Exception: `index.md` and `readme.md` at the root of each folder remain lowercase

### Documentation Structure

- Every markdown file **must** start with a breadcrumb: `[Documentation Index](../index.md) / [module] / [topic]`
- Root-level files use: `[Documentation Index](./index.md)`
- Use semantic headers (`#`, `##`, `###`, `####`)
- API endpoint docs must include: method, path, auth requirement, request shape, response shape, error codes

### AI Formatting Constraints

- **Diagrams:** Use Mermaid.js (`sequenceDiagram`, `flowchart`) for flows and architecture
- **Code:** Wrap all snippets in triple backticks with the correct language tag (`bash`, `typescript`, `json`)
- **Alerts:** Use markdown callouts (`> [!WARNING]`, `> [!NOTE]`) for critical constraints

---

## 🛠 Prohibitions (The "Never" List)

1. **NEVER** expose real API keys, tokens, Stripe secrets, or credentials — use `[REDACTED_API_KEY]`, `[STRIPE_SECRET]`, `[WEBHOOK_SECRET]`
2. **NEVER** create fragmented documentation — if a feature changes, update the existing `.md` file, never create `v2` or `notes` variants
3. **NEVER** skip updating `docs/module6-docMaintenance/documentationProgress.md` after any documentation session
4. **NEVER** skip updating `changelog.md` and `changelog.json` after a documentation session
5. **NEVER** document behaviour that contradicts the source code — read the file before documenting it

---

## 🗺 Directory Mapping (Context Hints)

- **`src/routes/`** → Express routers — one file per resource (`orders.ts`, `payments.ts`, `webhooks.ts`)
- **`src/services/`** → Business logic — `OrderService.ts`, `PaymentService.ts`, `NotificationService.ts`
- **`src/models/`** → Mongoose schemas — `Order.ts`, `Payment.ts`
- **`src/middleware/`** → Auth guard (`requireAuth.ts`), idempotency check (`idempotency.ts`), webhook signature verifier (`verifyWebhook.ts`)
- **`src/validators/`** → Zod request validators — one file per route group
- **`tests/`** → Integration tests using real MongoDB (no mocks on the database layer)
- **`infra/`** → Terraform for AWS infrastructure, GitHub Actions workflows

---

## ⚙️ Critical System Behaviours (AI Must Know)

### Order Lifecycle

```
pending → confirmed → processing → fulfilled → delivered
                   ↘ cancelled
         ↘ refunded (from confirmed or processing only)
```

State transitions are validated in `OrderService.ts`. Illegal transitions throw a `400 Bad Request` with code `INVALID_STATE_TRANSITION`.

### Idempotency

All `POST` requests to `/orders` and `/payments` require an `Idempotency-Key` header (UUID v4). Duplicate keys within 24 hours return the original response without re-executing the operation. Implemented in `src/middleware/idempotency.ts`.

### Webhook Verification

Inbound webhooks from fulfilment partners must pass HMAC-SHA256 signature verification against the raw request body. Implemented in `src/middleware/verifyWebhook.ts`. **Never skip this middleware on webhook routes.**

### Integration Tests Only

Tests in `tests/` connect to a real MongoDB test database — no mocking of the database layer. This is a hard rule following an incident where mock-based tests passed while a production migration failed.
