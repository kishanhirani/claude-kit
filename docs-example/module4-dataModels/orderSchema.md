[Documentation Index](../index.md) / module4-dataModels / Order Schema

# Order Schema

**File:** `src/models/Order.ts`
**Collection:** `orders`

---

## Schema Fields

| Field | Type | Required | Default | Constraints |
|---|---|---|---|---|
| `_id` | ObjectId | Auto | Auto | MongoDB default |
| `customerId` | ObjectId (ref: User) | Yes | — | Indexed |
| `status` | string (enum) | Yes | `pending` | `OrderStatus` enum |
| `items` | Item[] | Yes | — | Min length: 1 |
| `totalAmount` | number | Yes | — | Integer, > 0, pence/cents |
| `currency` | string | Yes | — | ISO 4217, e.g. `GBP`, `USD` |
| `shippingAddress` | Address | Yes | — | Nested object, all sub-fields required |
| `fulfilmentPartnerId` | ObjectId (ref: Partner) | Yes | — | Must reference active partner |
| `fulfilmentStatus` | string (enum) | No | `pending` | `FulfilmentStatus` enum |
| `trackingNumber` | string | No | — | Set by fulfilment webhook |
| `cancellationReason` | string | No | — | Required if status is `cancelled` |
| `refundReason` | string | No | — | Required if status is `refunded` |
| `createdAt` | Date | Auto | Auto | Managed by `timestamps: true` |
| `updatedAt` | Date | Auto | Auto | Managed by `timestamps: true` |

---

## OrderStatus Enum

```typescript
export const OrderStatus = {
  PENDING: 'pending',
  CONFIRMED: 'confirmed',
  PROCESSING: 'processing',
  FULFILLED: 'fulfilled',
  DELIVERED: 'delivered',
  CANCELLED: 'cancelled',
  REFUNDED: 'refunded',
} as const;
```

---

## Item Sub-document

| Field | Type | Required | Constraints |
|---|---|---|---|
| `productId` | ObjectId | Yes | — |
| `quantity` | number | Yes | Integer, >= 1 |
| `unitPrice` | number | Yes | Integer, > 0, pence/cents |
| `name` | string | Yes | Snapshot of product name at order time |

> [!WARNING]
> `unitPrice` and `quantity` are snapshotted at order creation time. They are never updated retroactively even if the product price changes. This is intentional — the order record is an immutable receipt of what was agreed at purchase.

---

## Address Sub-document

| Field | Type | Required |
|---|---|---|
| `line1` | string | Yes |
| `line2` | string | No |
| `city` | string | Yes |
| `postcode` | string | Yes |
| `country` | string | Yes (ISO 3166-1 alpha-2) |

---

## Indexes

```typescript
OrderSchema.index({ customerId: 1, createdAt: -1 });
OrderSchema.index({ status: 1 });
OrderSchema.index({ fulfilmentPartnerId: 1 });
```

---

## Schema Definition (excerpt)

```typescript
const OrderSchema = new Schema<IOrder>(
  {
    customerId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
    status: { type: String, enum: Object.values(OrderStatus), default: OrderStatus.PENDING },
    items: {
      type: [ItemSchema],
      validate: { validator: (v: unknown[]) => v.length > 0, message: 'Order must have at least one item' },
    },
    totalAmount: { type: Number, required: true, min: 1 },
    currency: { type: String, required: true },
    shippingAddress: { type: AddressSchema, required: true },
    fulfilmentPartnerId: { type: Schema.Types.ObjectId, ref: 'Partner', required: true },
    fulfilmentStatus: { type: String, enum: Object.values(FulfilmentStatus), default: FulfilmentStatus.PENDING },
    trackingNumber: { type: String },
    cancellationReason: { type: String },
    refundReason: { type: String },
  },
  { timestamps: true }
);
```
