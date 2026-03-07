---
name: init
description: Scaffold a complete HitPay payment integration in your project. Detects framework, creates API routes, webhook handler, and frontend component.
user_invocable: true
---

# /hitpay:init

Scaffold a complete HitPay payment integration. Supports `$ARGUMENTS` for quick setup (e.g., `/hitpay:init nextjs sg,my`).

## Instructions

Follow these steps in order:

### 1. Detect Framework

Check the project for framework signals:
- `next.config.*` or `app/` directory → **Next.js (App Router)**
- `pages/` directory with `next.config.*` → **Next.js (Pages Router)**
- `express` in package.json → **Express.js**
- `package.json` exists but no framework → **Node.js**
- No `package.json` → Ask the developer what framework they're using

If `$ARGUMENTS` includes a framework name (e.g., `nextjs`, `express`), use that instead of detecting.

### 2. Ask Developer Preferences

If not provided via `$ARGUMENTS`, ask:

1. **Which markets?** (e.g., SG, MY, PH, TH — determines payment methods)
2. **Which flow?** Redirect (cards), Embedded QR, Drop-In UI, or all three?
3. **Currency?** (default based on market: SG→SGD, MY→MYR, PH→PHP, etc.)

Parse `$ARGUMENTS` if provided:
- `/hitpay:init nextjs sg` → Next.js, Singapore (SGD, PayNow + cards)
- `/hitpay:init express sg,my` → Express.js, Singapore + Malaysia

### 3. Create Environment File

Create `.env.local` (Next.js) or `.env` (Express/Node) with placeholders:

```
# HitPay API Configuration
# Get your keys from: https://dashboard.sandbox.hit-pay.com/settings/payment-gateway/api-keys
HITPAY_API_KEY=your_sandbox_api_key_here
HITPAY_SALT=your_webhook_salt_here
HITPAY_ENV=sandbox
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_HITPAY_ENV=sandbox
```

Add `HITPAY_API_KEY` and `HITPAY_SALT` to `.env.example` if it exists (without real values).

### 4. Create API Routes

Use the **payment-integration** skill patterns to create:

1. **Payment creation route** — Creates a payment request
2. **Webhook handler route** — Verifies signature, processes payment events (use **webhook-handler** skill patterns)
3. **Payment status route** — Checks payment request status

For Next.js App Router:
- `app/api/payments/create/route.ts`
- `app/api/webhooks/hitpay/route.ts`
- `app/api/payments/[id]/route.ts`

For Express.js:
- `routes/payments.ts`
- `routes/webhooks.ts`

### 5. Create Frontend Component

Based on the chosen flow:
- **Redirect**: Create a `CheckoutButton` component
- **Embedded QR**: Create a `QRPayment` component (install `qrcode` package)
- **Drop-In UI**: Create a `DropInCheckout` component (add HitPay.js script, use **drop-in-ui** skill patterns)
- **All three**: Create a `PaymentMethodSelector` that routes to the appropriate flow

### 6. Install Dependencies

If QR flow was selected:
```bash
npm install qrcode
npm install -D @types/qrcode
```

### 7. Provide Quick-Start Guide

After scaffolding, output:

```
HitPay integration scaffolded!

Next steps:
1. Get your sandbox API keys from https://dashboard.sandbox.hit-pay.com
2. Update .env.local with your HITPAY_API_KEY and HITPAY_SALT
3. Start your dev server and test with:
   - Test card: 4242 4242 4242 4242 (any expiry, any CVC)
   - PayNow: Use sandbox QR scanner
4. When ready for production, update HITPAY_ENV=production and use production keys

Files created:
- [list all created files]

Commands:
- /hitpay:methods — Look up available payment methods
- /hitpay:webhook-test — Generate test webhook payloads
```
