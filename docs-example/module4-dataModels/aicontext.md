[Documentation Index](../index.md) / module4-dataModels / AI Context

# AI Context: Data Models

This file provides localised context for AI assistants working on the Data Models documentation section.

---

## 🏗 Section Identity

The **Data Models** section documents MongoDB schemas — fields, types, constraints, indexes, and the rules the application enforces on top of the database. AI assistants must read `src/models/` source files before proposing any schema changes or documentation updates.

---

## 📚 Key Terminology

- **ObjectId:** MongoDB's default `_id` type. Exposed as a hex string in API responses via `toJSON()` transform.
- **Monetary integer:** All price and amount fields are stored as integers (pence/cents). `unitPrice: 2999` means £29.99. Never store as a float.
- **Enum constraint:** Status fields are restricted to a TypeScript `const` enum. Both the enum and the Mongoose schema must be kept in sync — the enum is the source of truth.
- **`timestamps: true`:** Applied to all schemas. Mongoose automatically manages `createdAt` and `updatedAt`. Never add these fields manually to a schema.
- **Compound index:** An index on multiple fields in a defined order. Field order matters — a compound index on `(customerId, createdAt)` supports queries filtering by `customerId` but not queries filtering by `createdAt` alone.

---

## 💾 Core File Affiliation

- `src/models/Order.ts` — Order Mongoose model and schema
- `src/models/Payment.ts` — Payment Mongoose model and schema
- `src/types/enums.ts` — `OrderStatus`, `PaymentStatus` enum definitions

---

## 🖇 Mandatory Cross-References

- `docs/module3-services/orderService.md` — how OrderService reads and writes Order documents
- `docs/module3-services/paymentService.md` — how PaymentService reads and writes Payment documents
- `docs/module2-api/ordersEndpoints.md` — API fields that map to Order schema fields

---

## 🎨 Formatting Rules

- Use a table for all schema fields: `Field | Type | Required | Default | Constraints`
- Use `typescript` code blocks for schema definition excerpts
- Use `> [!WARNING]` for any field where the storage format differs from the API format (e.g. monetary integers)

---

## 🚨 Strict Constraints

1. Never suggest storing monetary values as floats — always integers in smallest currency unit
2. Never suggest adding `createdAt` or `updatedAt` fields manually — `timestamps: true` handles them
3. Never suggest mocking MongoDB in tests — integration tests use a real test database
4. Never change an enum value without updating both `src/types/enums.ts` and the Mongoose schema `enum:` array
