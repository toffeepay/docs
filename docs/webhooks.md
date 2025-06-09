---
sidebar_position: 10
---

# Webhooks

ToffeePay uses webhooks to notify your backend when important events occur, such as successful payments and completed refunds. All webhooks are signed for security.

## Webhook Structure

All ToffeePay webhooks follow a consistent structure with four key fields:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "event": "event.type",
  "timestamp": "2023-06-01T12:05:00Z",
  "data": {
    // Event-specific data
  }
}
```

**Fields:**
- **`id`**: Unique webhook identifier for idempotency handling
- **`event`**: Event type (e.g., `payment.succeeded`, `refund.succeeded`)
- **`timestamp`**: ISO 8601 timestamp when the event occurred
- **`data`**: Event-specific payload containing relevant entity data

---

## Webhook Events

ToffeePay sends webhooks for various events throughout the payment lifecycle:

- **Payment Events**: See the [Payments](/payments#webhook-events) page for session and payment events
- **Refund Events**: See the [Refunds](/refunds#webhook-events) page for refund events

---

## Webhook Signature Verification

All webhooks include a signature in the `X-ToffeePay-Signature` header that you must verify to ensure the webhook is authentic.

**Header Format:**
```
X-ToffeePay-Signature: t=1717000000,v1=6aeedfabc12345...
```

### Verification Steps

1. **Parse the signature header**: Extract the timestamp (`t`) and signature (`v1`) from the header
2. **Create the signed payload**: Concatenate the timestamp, a period, and the raw request body:
   ```
   signed_payload = t + "." + raw_body
   ```
3. **Compute the expected signature**: Use HMAC SHA256 with your webhook secret:
   ```
   expected_signature = HMAC_SHA256(your_webhook_secret, signed_payload)
   ```
4. **Validate the timestamp**: Ensure `t` is within 5 minutes of the current time to prevent replay attacks
5. **Compare signatures**: Use a constant-time comparison to compare the expected signature with `v1`

### Example Verification (TypeScript)

```typescript
import { createHmac, timingSafeEqual } from 'crypto';

interface Webhook {
  id: string;
  event: string;
  timestamp: string;
  data: any;
}

function verifyWebhook(
  rawPayload: string, 
  signatureHeader: string, 
  webhookSecret: string
): boolean {
  // Parse signature header
  const elements = signatureHeader.split(',');
  let timestamp: string | null = null;
  let signature: string | null = null;
  
  for (const element of elements) {
    if (element.startsWith('t=')) {
      timestamp = element.slice(2);
    } else if (element.startsWith('v1=')) {
      signature = element.slice(3);
    }
  }
  
  if (!timestamp || !signature) {
    return false;
  }
  
  // Parse webhook data to verify timestamp matches
  const webhookData: Webhook = JSON.parse(rawPayload);
  
  // Check timestamp (within 5 minutes)
  const currentTime = Math.floor(Date.now() / 1000);
  const webhookTime = Math.floor(new Date(webhookData.timestamp).getTime() / 1000);
  if (Math.abs(currentTime - parseInt(timestamp)) > 300) {
    return false;
  }
  
  // Create signed payload
  const signedPayload = `${timestamp}.${rawPayload}`;
  
  // Compute expected signature
  const expectedSignature = createHmac('sha256', webhookSecret)
    .update(signedPayload, 'utf8')
    .digest('hex');
  
  // Compare signatures (constant-time)
  const expectedBuffer = Buffer.from(expectedSignature, 'hex');
  const signatureBuffer = Buffer.from(signature, 'hex');
  
  return expectedBuffer.length === signatureBuffer.length && 
         timingSafeEqual(expectedBuffer, signatureBuffer);
}

// Usage example
function handleWebhook(req: Request) {
  const rawPayload = req.body;
  const signature = req.headers['x-toffeepay-signature'];
  
  if (!verifyWebhook(rawPayload, signature, process.env.WEBHOOK_SECRET)) {
    throw new Error('Invalid webhook signature');
  }
  
  const webhook: Webhook = JSON.parse(rawPayload);
  
  switch (webhook.event) {
    case 'payment.succeeded':
      handlePaymentSuccess(webhook.data);
      break;
    case 'refund.succeeded':
      handleRefundSuccess(webhook.data);
      break;
    // ... other events
  }
}
```

---

## Webhook Configuration

You can configure and manage your webhook settings:

1. **Webhook URL**: The endpoint where ToffeePay will send webhooks
2. **Webhook Secret**: Used to sign webhook payloads (can be rotated)
3. **Event Selection**: Choose which events to receive (payments, refunds, or both)

---

## Best Practices

1. **Always verify signatures**: Never process webhooks without signature verification
2. **Handle duplicate events**: While rare, webhooks may be sent multiple times, implement [idempotency](/idempotency#webhook-idempotency) using webhook IDs
3. **Respond quickly**: Return a 200 status code within 10 seconds to acknowledge receipt
4. **Retry logic**: ToffeePay will retry failed webhooks, but implement your own retry logic for critical operations
5. **Log webhook events**: Keep audit logs of all webhook events for debugging and compliance
6. **Use HTTPS**: Always use HTTPS endpoints for webhook URLs

---

## Troubleshooting

### Common Issues

- **Signature verification fails**: Check that you're using the correct webhook secret and following the verification steps exactly
- **Timeout errors**: Ensure your webhook endpoint responds within 10 seconds
- **Missing webhooks**: Check your webhook URL configuration and ensure your endpoint is accessible

### Testing Webhooks

Use tools like ngrok for local development to expose your local webhook endpoint to ToffeePay:

```bash
ngrok http 3000
```