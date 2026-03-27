[Documentation Index](../index.md) / module5-workflows / AI Context

# AI Context: Workflows

This file provides localised context for AI assistants working on the Workflows documentation section.

---

## 🏗 Section Identity

The **Workflows** section documents end-to-end flows across multiple services and API layers. It is not an API reference — it is a narrative description of how the system behaves across time. AI assistants editing this section must be consistent with the state machine in `aicontext.md` and the service behaviour in `module3-services/`.

---

## 📚 Key Terminology

- **Happy path:** The flow where all operations succeed — payment captured, order routed, delivered. The baseline against which all failure scenarios are defined.
- **Routing failure:** When `OrderService.routeToFulfilmentPartner()` cannot reach the partner. Does not roll back order status. Requires manual ops intervention.
- **Partial refund:** A Stripe refund for less than the full charge amount. Not currently supported — received and logged only.
- **Retry:** In the payment failure workflow, a retry requires a new idempotency key and may use a different payment method.

---

## 💾 Core File Affiliation

- `src/services/OrderService.ts` — state transitions and routing logic documented in these workflows
- `src/services/PaymentService.ts` — payment success/failure propagation
- `src/services/NotificationService.ts` — notification events emitted at each workflow step

---

## 🖇 Mandatory Cross-References

- `docs/module2-api/ordersEndpoints.md` — the API calls made in each workflow step
- `docs/module2-api/webhooksEndpoints.md` — inbound webhook events that drive workflow progression
- `docs/module3-services/orderService.md` — service behaviour behind each workflow step

---

## 🎨 Formatting Rules

- Use Mermaid `sequenceDiagram` for all multi-step flows involving external systems (Stripe, partners)
- Use numbered steps for flows that are purely internal (no external participant)
- Use `> [!WARNING]` for any flow that has a non-obvious consequence (e.g. cancellation does not auto-refund)
- Use `> [!NOTE]` for clarifications that prevent common integration mistakes

---

## 🚨 Strict Constraints

1. Never describe a workflow step that contradicts the state machine — illegal transitions must not appear as valid steps
2. Never imply that notification failures block the workflow — notifications are always fire-and-forget
3. Never suggest that partial refunds are handled — they are explicitly not supported and must not be documented as working
4. Never omit the idempotency key requirement from payment workflow steps — it is mandatory and a common integration mistake
