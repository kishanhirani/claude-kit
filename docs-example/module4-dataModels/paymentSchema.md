[Documentation Index](../index.md) / module4-dataModels / Payment Schema

# Payment Schema

**File:** `src/models/Payment.ts`
**Collection:** `payments`

---

## Schema Fields

| Field | Type | Required | Default | Constraints |
|---|---|---|---|---|
| `_id` | ObjectId | Auto | Auto | MongoDB default |
| `orderId` | ObjectId (ref: Order) | Yes | — | Unique — one payment per order |
| `customerId` | ObjectId (ref: User) | Yes | — | Denormalised for query convenience |
| `stripePaymentIntentId` | string | Yes | — | Unique, Stripe `pi_` prefix |
| `status` | string (enum) | Yes | `pending` | `PaymentStatus` enum |
| `amount` | number | Yes | — | Integer, > 0, pence/cents |
| `currency` | string | Yes | — | ISO 4217 |
| `failureReason` | string | No | — | Populated from Stripe event on failure |
| `refundedAt` | Date | No | — | Set when `charge.refunded` webhook received |
| `createdAt` | Date | Auto | Auto | Managed by `timestamps: true` |
| `updatedAt` | Date | Auto | Auto | Managed by `timestamps: true` |

---

## PaymentStatus Enum

```typescript
export const PaymentStatus = {
  PENDING: 'pending',
  SUCCEEDED: 'succeeded',
  FAILED: 'failed',
  REFUNDED: 'refunded',
} as const;
```

---

## Indexes

```typescript
PaymentSchema.index({ orderId: 1 }, { unique: true });
PaymentSchema.index({ stripePaymentIntentId: 1 }, { unique: true });
PaymentSchema.index({ customerId: 1 });
```

The unique index on `orderId` enforces the one-payment-per-order constraint at the database layer in addition to the application-layer check in `PaymentService`.

---

## Schema Definition (excerpt)

```typescript
const PaymentSchema = new Schema<IPayment>(
  {
    orderId: { type: Schema.Types.ObjectId, ref: 'Order', required: true },
    customerId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
    stripePaymentIntentId: { type: String, required: true },
    status: { type: String, enum: Object.values(PaymentStatus), default: PaymentStatus.PENDING },
    amount: { type: Number, required: true, min: 1 },
    currency: { type: String, required: true },
    failureReason: { type: String },
    refundedAt: { type: Date },
  },
  { timestamps: true }
);
```

> [!WARNING]
> `customerId` is denormalised onto the Payment document to avoid a join when checking payment ownership in `PaymentService.getById()`. If an order's customer ever changes (not a supported operation in the current system), this field would become stale. Do not add customer-change functionality without also updating this field.
