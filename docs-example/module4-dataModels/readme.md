[Documentation Index](../index.md) / module4-dataModels

# Data Models Overview

Orderflow uses MongoDB 7 with Mongoose 8. Schemas are defined in `src/models/` and enforce field-level constraints, indexes, and validation at the application layer.

---

## Table of Contents

- [Conventions](#conventions)
- [Models](#models)
- [Index Strategy](#index-strategy)
- [Detailed Schema Documentation](#detailed-schema-documentation)

---

## Conventions

- All documents include `createdAt` and `updatedAt` via Mongoose `timestamps: true`
- All IDs are MongoDB `ObjectId`, exposed as strings in API responses
- Monetary amounts are stored as integers in the smallest currency unit (pence / cents) — never floats
- Soft deletes are not used — documents are hard deleted when required
- Enum fields use TypeScript `const` enums defined in `src/types/enums.ts` — the same values are used in Mongoose schemas as `enum: Object.values(MyEnum)`

---

## Models

| Model | File | Collection |
|---|---|---|
| Order | `src/models/Order.ts` | `orders` |
| Payment | `src/models/Payment.ts` | `payments` |

---

## Index Strategy

Indexes are created automatically by Mongoose on application start. The index strategy prioritises the most common query patterns:

| Collection | Index | Type | Reason |
|---|---|---|---|
| `orders` | `customerId + createdAt` | Compound | List orders for a customer, sorted by date |
| `orders` | `status` | Single | Filter by status |
| `orders` | `fulfilmentPartnerId` | Single | Routing queries |
| `payments` | `orderId` | Single, Unique | One payment per order |
| `payments` | `stripePaymentIntentId` | Single, Unique | Stripe webhook lookup |

---

## Detailed Schema Documentation

- [Order Schema](./orderSchema.md)
- [Payment Schema](./paymentSchema.md)
