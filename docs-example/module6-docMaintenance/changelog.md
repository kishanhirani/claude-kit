[Documentation Index](../index.md) / module6-docMaintenance / Changelog

# Changelog

All documentation sessions are recorded here, newest first.

---

| Session | Date | Focus | Summary |
|---|---|---|---|
| 6 | 2026-01-28 | Refund workflow and partial refund clarification | Added refund workflow to module5-workflows covering the Stripe-initiated refund flow, documented partial refund non-support explicitly in PaymentService and webhooks, and clarified that charge.refunded events for partial amounts are logged but produce no status change. |
| 5 | 2026-01-20 | Fulfilment partner webhook events and routing failure handling | Documented the fulfilment partner webhook endpoint in full including all supported event types and their order status transitions, added routing failure behaviour to OrderService docs, and added the fulfilment partner dispatch workflow scenario. |
| 4 | 2026-01-15 | Payments endpoint, PaymentService, and Payment schema | Created full documentation for the payments API endpoint, PaymentService methods (createIntent, syncFromStripeEvent, getById), and the Payment Mongoose schema including the unique orderId index, PaymentStatus enum, and denormalisation rationale for customerId. |
| 3 | 2026-01-10 | Orders endpoints, OrderService, and Order schema | Created full documentation for all four orders API endpoints (create, get, patch status, list), the OrderService state machine and all methods, and the Order Mongoose schema including Item and Address sub-documents, enum values, and index definitions. |
| 2 | 2025-11-14 | Services layer, data models overview, and workflows foundation | Documented the services architecture overview with Mermaid flowchart, NotificationService in full (emit pattern, all event types, SMTP config, fire-and-forget failure behaviour), data models conventions and index strategy, and the happy path and payment failure workflow scenarios. |
| 1 | 2025-10-01 | Initial documentation structure — global context, getting started, API overview | Created the complete initial documentation structure for Orderflow: master index, global AI context (order lifecycle, idempotency, webhook verification, integration-test-only rule), getting started guide with full environment variable reference, API overview with authentication and error format, and the doc-maintenance system. |
