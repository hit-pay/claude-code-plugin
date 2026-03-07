---
name: webhook-handler
description: Handle HitPay payment webhooks with signature verification. Use when user says "HitPay webhook", "verify webhook signature", "payment notification", "webhook handler", "Hitpay-Signature", or "payment confirmation".
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay Webhook Handler

Complete guide to receiving and verifying HitPay payment webhooks. Covers v2 webhook headers, HMAC-SHA256 signature verification, event types, IP allowlisting, and idempotent processing.

## When to Apply

- Setting up webhook endpoints for HitPay payments
- Verifying `Hitpay-Signature` headers
- Processing payment completion/failure notifications
- Debugging webhook delivery issues

> **Need the full payment integration?** See the **payment-integration** skill.
> **Want to test webhooks locally?** Use the `/hitpay:webhook-test` command.

## Webhook Headers (v2)

| Header | Description |
|--------|-------------|
| `Hitpay-Signature` | HMAC-SHA256 hash of the JSON payload |
| `Hitpay-Event-Type` | `created` or `updated` |
| `Hitpay-Event-Object` | Object type: `charge`, `payment_request`, `payout`, etc. |
| `User-Agent` | `HitPay v2.0` |
| `Content-Type` | `application/json` |

## Event Types

### Payment Request Events

| Event | Description |
|-------|-------------|
| `payment_request.completed` | Payment was successful |
| `payment_request.failed` | Payment failed |

### Charge Events

| Event | Description |
|-------|-------------|
| `charge.created` | New charge created |
| `charge.updated` | Charge status updated |

### Other Events

| Event | Description |
|-------|-------------|
| `payout.created` | Payout initiated |
| `invoice.created` | Invoice created |
| `order.created` | Order created |
| `order.updated` | Order updated |
| `transfer.created` | Transfer created |
| `transfer.paid` | Transfer completed |
| `transfer.failed` | Transfer failed |

## Signature Verification

### How It Works

1. HitPay sends the webhook with a `Hitpay-Signature` header
2. The signature is an HMAC-SHA256 hash of the raw request body
3. The secret key is your **salt** from the HitPay dashboard (Settings > Developers > Webhook Endpoints)
4. Compare your computed signature with the header value using timing-safe comparison

### Next.js App Router Implementation

```typescript
// app/api/webhooks/hitpay/route.ts
import { NextResponse } from 'next/server';
import crypto from 'crypto';

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get('Hitpay-Signature');
  const eventType = request.headers.get('Hitpay-Event-Type');
  const eventObject = request.headers.get('Hitpay-Event-Object');

  // Verify signature using HMAC-SHA256
  const expectedSignature = crypto
    .createHmac('sha256', process.env.HITPAY_SALT!)
    .update(body)
    .digest('hex');

  if (signature !== expectedSignature) {
    console.error('Invalid webhook signature');
    return NextResponse.json({ error: 'Invalid signature' }, { status: 401 });
  }

  const payload = JSON.parse(body);

  // Handle different event types
  switch (`${eventObject}.${eventType}`) {
    case 'payment_request.completed':
      await handlePaymentCompleted(payload);
      break;
    case 'payment_request.failed':
      await handlePaymentFailed(payload);
      break;
    default:
      console.log(`Unhandled event: ${eventObject}.${eventType}`);
  }

  return NextResponse.json({ received: true });
}

async function handlePaymentCompleted(payload: any) {
  const { reference_number, amount, currency } = payload;
  // Mark order as paid in your database
  console.log(`Payment completed: ${reference_number} - ${amount} ${currency}`);
}

async function handlePaymentFailed(payload: any) {
  const { reference_number } = payload;
  // Handle failed payment
  console.log(`Payment failed: ${reference_number}`);
}
```

### Express.js Implementation

```typescript
// routes/webhooks.ts
import express from 'express';
import crypto from 'crypto';

const router = express.Router();

// Important: Use raw body parser for webhook routes
router.post(
  '/hitpay',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const body = req.body.toString();
    const signature = req.headers['hitpay-signature'] as string;

    const expectedSignature = crypto
      .createHmac('sha256', process.env.HITPAY_SALT!)
      .update(body)
      .digest('hex');

    // Use timing-safe comparison to prevent timing attacks
    const isValid = crypto.timingSafeEqual(
      Buffer.from(signature || ''),
      Buffer.from(expectedSignature)
    );

    if (!isValid) {
      return res.status(401).json({ error: 'Invalid signature' });
    }

    const payload = JSON.parse(body);
    const eventObject = req.headers['hitpay-event-object'];
    const eventType = req.headers['hitpay-event-type'];

    console.log(`Received: ${eventObject}.${eventType}`, payload);
    res.json({ received: true });
  }
);

export default router;
```

### Reusable Utility Function

```typescript
// lib/hitpay.ts
import crypto from 'crypto';

export function verifyHitPaySignature(
  body: string,
  signature: string | null,
  salt: string
): boolean {
  if (!signature) return false;

  const expectedSignature = crypto
    .createHmac('sha256', salt)
    .update(body)
    .digest('hex');

  try {
    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(expectedSignature)
    );
  } catch {
    return false;
  }
}
```

## IP Allowlisting

For additional security, restrict webhook endpoints to HitPay's IP addresses:

| Environment | IP Addresses |
|-------------|-------------|
| Production | `3.1.13.32`, `52.77.254.34` |
| Sandbox | `54.179.156.147` |

### Next.js Middleware Example

```typescript
// middleware.ts (or inline in webhook route)
const HITPAY_IPS = {
  production: ['3.1.13.32', '52.77.254.34'],
  sandbox: ['54.179.156.147'],
};

function isHitPayIP(ip: string): boolean {
  const env = process.env.HITPAY_ENV === 'production' ? 'production' : 'sandbox';
  return HITPAY_IPS[env].includes(ip);
}
```

## Webhook Payload Examples

### Payment Request Completed

```json
{
  "id": "9ef68e2e-3569-4f69-9f68-04c7e4bb007c",
  "amount": "100.00",
  "currency": "sgd",
  "status": "completed",
  "reference_number": "ORDER-12345",
  "email": "customer@example.com",
  "name": "John Smith",
  "payment_type": "card",
  "payments": [
    {
      "id": "pay_abc123",
      "amount": "100.00",
      "currency": "sgd",
      "status": "succeeded"
    }
  ],
  "created_at": "2026-01-21T10:30:00",
  "updated_at": "2026-01-21T10:35:00"
}
```

### Payment Request Failed

```json
{
  "id": "9ef68e2e-3569-4f69-9f68-04c7e4bb007c",
  "amount": "100.00",
  "currency": "sgd",
  "status": "failed",
  "reference_number": "ORDER-12345",
  "failure_reason": "card_declined",
  "created_at": "2026-01-21T10:30:00",
  "updated_at": "2026-01-21T10:35:00"
}
```

## Setting Up Webhooks

### Via Dashboard (Recommended)

1. Go to HitPay Dashboard > Settings > Developers > Webhook Endpoints
2. Add your webhook URL (e.g., `https://yoursite.com/api/webhooks/hitpay`)
3. Select the events you want to receive
4. Copy the **salt** value for signature verification

### Via API (Per Request)

Include the `webhook` parameter when creating a payment request:

```typescript
{
  amount: 100,
  currency: 'SGD',
  webhook: 'https://yoursite.com/api/webhooks/hitpay',
}
```

**Note:** Dashboard webhooks are preferred over per-request webhooks for reliability.

## Best Practices

1. **Always verify signatures** — Never process webhooks without HMAC verification
2. **Use HTTPS** — Webhook URLs must use TLS
3. **Return 200 quickly** — Process asynchronously if needed; HitPay expects a response within 30 seconds
4. **Handle duplicates** — HitPay may retry failed deliveries; use idempotency checks
5. **Log everything** — Keep webhook logs for debugging and reconciliation
6. **Never trust redirects alone** — Always confirm payment status via webhook before fulfilling orders

### Idempotent Processing

```typescript
async function handlePaymentCompleted(payload: any) {
  const { id, reference_number } = payload;

  // Check if already processed
  const existing = await db.webhookLogs.findUnique({ where: { paymentId: id } });
  if (existing) {
    console.log(`Webhook already processed: ${id}`);
    return;
  }

  // Process the payment
  await db.orders.update({
    where: { id: reference_number },
    data: { status: 'paid', paidAt: new Date() },
  });

  // Log the webhook
  await db.webhookLogs.create({
    data: { paymentId: id, processedAt: new Date() },
  });
}
```

## Environment Variables

```bash
# Get salt from HitPay Dashboard > Settings > Developers > Webhook Endpoints
HITPAY_SALT=your_webhook_salt_here
```
