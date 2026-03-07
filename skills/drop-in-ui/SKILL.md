---
name: drop-in-ui
description: Embed HitPay's Drop-In checkout UI (HitPay.js) into web applications. Use when user says "HitPay Drop-In", "embed payment form", "HitPay.js", "payment modal", "checkout popup", "inline checkout", or "embedded checkout".
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay Drop-In UI

Embed HitPay's hosted checkout as a modal overlay on your site using HitPay.js. The Drop-In handles payment method selection, card input, QR display, and 3DS — no PCI scope required.

## When to Apply

- Developer wants an embedded checkout without building custom UI
- Developer asks about HitPay.js, payment modal, or checkout popup
- Need a simpler alternative to building redirect or QR flows from scratch

> **Building custom flows?** See the **payment-integration** skill for redirect and embedded QR patterns.
> **Payment method lookup?** See the **payment-methods** skill.

## When to Use Drop-In vs Other Approaches

| Approach | Best For | PCI Scope | Customization |
|----------|----------|-----------|---------------|
| **Drop-In UI** | Quick integration, all-in-one checkout | None | Limited (colors, logo) |
| **Redirect** | Card payments, full HitPay-hosted checkout | None | None (HitPay page) |
| **Embedded QR** | QR-only payments, custom UI | None | Full control |

### Drop-In Limitations
- **Apple Pay**: Not supported in Drop-In — use redirect flow instead
- **Custom styling**: Limited to color and logo customization
- **Mobile**: Opens as full-screen overlay on mobile devices

## Step 1: Include HitPay.js

Add the script tag to your HTML or layout:

```html
<!-- Production -->
<script src="https://hit-pay.com/hitpay.js"></script>

<!-- Sandbox -->
<script src="https://sandbox.hit-pay.com/hitpay.js"></script>
```

For Next.js, add via `next/script`:

```typescript
// app/layout.tsx
import Script from 'next/script';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Script
          src={process.env.NEXT_PUBLIC_HITPAY_ENV === 'production'
            ? 'https://hit-pay.com/hitpay.js'
            : 'https://sandbox.hit-pay.com/hitpay.js'}
          strategy="afterInteractive"
        />
      </body>
    </html>
  );
}
```

## Step 2: Create Payment Request (Server)

Create a payment request on your server, then pass the URL to the client:

```typescript
// app/api/payments/create/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { amount, currency, orderId, email } = await request.json();

  const baseUrl = process.env.HITPAY_ENV === 'production'
    ? 'https://api.hit-pay.com'
    : 'https://api.sandbox.hit-pay.com';

  const response = await fetch(`${baseUrl}/v1/payment-requests`, {
    method: 'POST',
    headers: {
      'X-BUSINESS-API-KEY': process.env.HITPAY_API_KEY!,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      amount,
      currency,
      email,
      reference_number: orderId,
      redirect_url: `${process.env.NEXT_PUBLIC_APP_URL}/payment/complete`,
      webhook: `${process.env.NEXT_PUBLIC_APP_URL}/api/webhooks/hitpay`,
    }),
  });

  const data = await response.json();
  return NextResponse.json({
    paymentRequestId: data.id,
    paymentUrl: data.url,
  });
}
```

## Step 3: Initialize Drop-In (Client)

```typescript
// components/DropInCheckout.tsx
'use client';

import { useCallback } from 'react';

declare global {
  interface Window {
    HitPay: {
      init(url: string, options: HitPayInitOptions): void;
      toggle(options?: { open: boolean }): void;
    };
  }
}

interface HitPayInitOptions {
  domain?: string;
  apiDomain?: string;
  onSuccess?: (data: { payment_request_id: string; status: string; reference_number?: string }) => void;
  onError?: (error: { message: string }) => void;
  onClose?: () => void;
}

interface Props {
  amount: number;
  currency: string;
  orderId: string;
  email?: string;
}

export function DropInCheckout({ amount, currency, orderId, email }: Props) {
  const handleCheckout = useCallback(async () => {
    // 1. Create payment request on server
    const response = await fetch('/api/payments/create', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount, currency, orderId, email }),
    });

    const { paymentUrl } = await response.json();

    // 2. Initialize Drop-In with the payment URL
    window.HitPay.init(paymentUrl, {
      domain: process.env.NEXT_PUBLIC_HITPAY_ENV === 'production'
        ? 'hit-pay.com'
        : 'sandbox.hit-pay.com',
      apiDomain: process.env.NEXT_PUBLIC_HITPAY_ENV === 'production'
        ? 'api.hit-pay.com'
        : 'api.sandbox.hit-pay.com',
      onSuccess: (data) => {
        console.log('Payment successful:', data.payment_request_id);
        // Redirect to success page or update UI
        // IMPORTANT: Always verify via webhook before fulfilling
        window.location.href = `/payment/complete?ref=${data.reference_number}`;
      },
      onError: (error) => {
        console.error('Payment error:', error.message);
        // Show error to user
      },
      onClose: () => {
        console.log('Payment modal closed');
        // User closed without completing — no action needed
      },
    });

    // 3. Open the Drop-In modal
    window.HitPay.toggle({ open: true });
  }, [amount, currency, orderId, email]);

  return (
    <button onClick={handleCheckout}>
      Pay {currency} {amount.toFixed(2)}
    </button>
  );
}
```

## HitPay.init() Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `domain` | string | Yes | `'hit-pay.com'` (prod) or `'sandbox.hit-pay.com'` (sandbox) |
| `apiDomain` | string | Yes | `'api.hit-pay.com'` (prod) or `'api.sandbox.hit-pay.com'` (sandbox) |
| `onSuccess` | function | No | Called when payment completes successfully |
| `onError` | function | No | Called when payment fails |
| `onClose` | function | No | Called when user closes the modal |

### onSuccess Callback Data

```typescript
{
  payment_request_id: string;  // HitPay payment request ID
  status: string;              // "completed"
  reference_number?: string;   // Your order reference
}
```

### onError Callback Data

```typescript
{
  message: string;  // Human-readable error message
}
```

## Full Next.js Example

### Server Component (Page)

```typescript
// app/checkout/page.tsx
import { DropInCheckout } from '@/components/DropInCheckout';

export default function CheckoutPage({
  searchParams,
}: {
  searchParams: { orderId: string; amount: string };
}) {
  return (
    <main>
      <h1>Complete Your Purchase</h1>
      <DropInCheckout
        amount={parseFloat(searchParams.amount)}
        currency="SGD"
        orderId={searchParams.orderId}
      />
    </main>
  );
}
```

### Payment Complete Page

```typescript
// app/payment/complete/page.tsx
export default function PaymentCompletePage({
  searchParams,
}: {
  searchParams: { ref?: string };
}) {
  return (
    <main>
      <h1>Payment Received</h1>
      <p>Thank you! Your order {searchParams.ref} is being processed.</p>
      <p>You will receive a confirmation email shortly.</p>
    </main>
  );
}
```

## Important Notes

1. **Always verify via webhook** — The `onSuccess` callback is client-side and can be spoofed. Never fulfill orders based solely on the callback. See the **webhook-handler** skill.
2. **HitPay.js is loaded globally** — It attaches to `window.HitPay`. Only call `init()` after the script has loaded.
3. **One modal at a time** — Calling `init()` again replaces the previous payment session.
4. **Mobile behavior** — The Drop-In opens as a full-screen overlay on mobile devices.
5. **CSP headers** — If using Content Security Policy, allow `frame-src` for `hit-pay.com` or `sandbox.hit-pay.com`.

## Environment Variables

```bash
# .env.local
HITPAY_API_KEY=your_api_key
HITPAY_SALT=your_webhook_salt
HITPAY_ENV=sandbox  # or "production"
NEXT_PUBLIC_HITPAY_ENV=sandbox  # client-side env detection
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

See `references/dropin-ui-api.md` for the full HitPay.js API reference including CSS customization.
