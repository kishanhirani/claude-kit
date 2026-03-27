[Documentation Index](../index.md) / module2-api / Orders Endpoints

# Orders Endpoints

---

## POST /v1/orders

Create a new order.

**Auth:** Required
**Idempotency-Key:** Required

### Request Body

```json
{
  "customerId": "[CUSTOMER_ID]",
  "items": [
    {
      "productId": "[PRODUCT_ID]",
      "quantity": 2,
      "unitPrice": 2999
    }
  ],
  "shippingAddress": {
    "line1": "12 Example Street",
    "city": "London",
    "postcode": "EC1A 1BB",
    "country": "GB"
  },
  "fulfilmentPartnerId": "[PARTNER_ID]"
}
```

- `unitPrice` is in the smallest currency unit (pence / cents)
- `fulfilmentPartnerId` must reference an active partner in the system

### Response — 201 Created

```json
{
  "id": "[ORDER_ID]",
  "status": "pending",
  "customerId": "[CUSTOMER_ID]",
  "items": [...],
  "totalAmount": 5998,
  "currency": "GBP",
  "createdAt": "2026-01-15T09:00:00.000Z"
}
```

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `400` | `MISSING_IDEMPOTENCY_KEY` | `Idempotency-Key` header absent |
| `400` | `VALIDATION_ERROR` | Request body fails Zod schema (field set in `error.field`) |
| `404` | `PARTNER_NOT_FOUND` | `fulfilmentPartnerId` does not match an active partner |
| `409` | `DUPLICATE_IDEMPOTENCY_KEY` | Key used with a different request body within 24h |
| `401` | `TOKEN_EXPIRED` | JWT has expired |

---

## GET /v1/orders/:id

Get a single order by ID.

**Auth:** Required

### Response — 200 OK

```json
{
  "id": "[ORDER_ID]",
  "status": "processing",
  "customerId": "[CUSTOMER_ID]",
  "items": [...],
  "totalAmount": 5998,
  "currency": "GBP",
  "payment": {
    "id": "[PAYMENT_ID]",
    "status": "succeeded"
  },
  "createdAt": "2026-01-15T09:00:00.000Z",
  "updatedAt": "2026-01-15T09:05:00.000Z"
}
```

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `404` | `ORDER_NOT_FOUND` | No order with this ID exists |
| `403` | `FORBIDDEN` | Order belongs to a different customer |

---

## PATCH /v1/orders/:id/status

Transition an order to a new status.

**Auth:** Required

### Request Body

```json
{
  "status": "cancelled",
  "reason": "Customer requested cancellation"
}
```

- `reason` is required when transitioning to `cancelled` or `refunded`
- Valid transitions: see the order lifecycle state machine in [Global AI Context](../aicontext.md)

### Response — 200 OK

```json
{
  "id": "[ORDER_ID]",
  "status": "cancelled",
  "updatedAt": "2026-01-15T09:10:00.000Z"
}
```

### Error Codes

| HTTP Status | Code | Trigger |
|---|---|---|
| `400` | `INVALID_STATE_TRANSITION` | Requested status is not reachable from current status |
| `400` | `REASON_REQUIRED` | `reason` missing on cancellation or refund |
| `404` | `ORDER_NOT_FOUND` | No order with this ID exists |

---

## GET /v1/orders

List orders for the authenticated customer, paginated.

**Auth:** Required

### Query Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number (1-indexed) |
| `limit` | integer | `20` | Items per page (max 100) |
| `status` | string | none | Filter by order status |
| `from` | ISO 8601 date | none | Filter orders created after this date |
| `to` | ISO 8601 date | none | Filter orders created before this date |

### Response — 200 OK

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 143,
    "totalPages": 8
  }
}
```
