---
sidebar_position: 3
---

# Idempotency

Idempotency ensures that retrying the same request multiple times has the same effect as making it once. This is crucial for preventing duplicate payments, refunds, and other operations in unreliable network conditions.

## Idempotency-Key Header

ToffeePay supports idempotency for mutation operations (POST requests) using the `Idempotency-Key` header:

```http
POST /pay.v1.PaymentService/CreateSession
Authorization: Bearer <your_api_key>
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

...

```

## How It Works

1. **First Request**: ToffeePay processes the request and stores the result with the idempotency key
2. **Duplicate Requests**: If the same idempotency key is used again within 24 hours, ToffeePay returns the cached result instead of processing again
3. **Different Parameters**: If the same key is used with different request parameters, ToffeePay returns an error

## Supported Operations

Idempotency is supported for these operations:

- **CreateSession** - Prevents duplicate payment sessions
- **CreateRefund** - Prevents duplicate refunds
- Any other mutation operations that could have side effects

## Best Practices

### Generate Unique Keys
Use UUIDs or other collision-resistant identifiers:

```typescript
import { v4 as uuidv4 } from 'uuid';

const idempotencyKey = uuidv4(); // e.g., "550e8400-e29b-41d4-a716-446655440000"
```

### Key Per Operation
Use a different idempotency key for each distinct operation:

```typescript
// Good: Different keys for different operations
const sessionKey = `session-${userId}-${itemId}-${Date.now()}`;
const refundKey = `refund-${paymentId}-${Date.now()}`;

// Bad: Reusing the same key
const key = "my-operation-key"; // Don't do this
```

## Error Handling

### Idempotency Conflicts
If you use the same idempotency key with different parameters:

```http
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "error": {
    "code": "idempotency_conflict",
    "message": "Idempotency key already used with different parameters"
  }
}
```

### Key Expiration
Idempotency keys expire after 24 hours. After expiration, the same key can be reused.

## Webhook Idempotency

For webhook processing, use the webhook's unique `id` field for idempotency:

```typescript
const processedWebhooks = new Set<string>();

function handleWebhook(webhook: Webhook) {
  // Check if we've already processed this webhook
  if (processedWebhooks.has(webhook.id)) {
    console.log('Webhook already processed:', webhook.id);
    return; // Skip processing
  }
  
  // Process the webhook
  switch (webhook.event) {
    case 'payment.succeeded':
      handlePaymentSuccess(webhook.data);
      break;
    // ... other events
  }
  
  // Mark as processed
  processedWebhooks.add(webhook.id);
}
```

---

## Summary

- Use `Idempotency-Key` header for all mutation operations
- Generate unique UUIDs for each operation
- Handle idempotency conflicts gracefully
- Use webhook IDs for webhook idempotency
- Keys expire after 24 hours

Proper idempotency implementation ensures reliable payment processing even in unreliable network conditions.