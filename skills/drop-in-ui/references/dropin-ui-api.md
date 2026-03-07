# HitPay.js Drop-In UI API Reference

## Script URLs

| Environment | URL |
|-------------|-----|
| Production | `https://hit-pay.com/hitpay.js` |
| Sandbox | `https://sandbox.hit-pay.com/hitpay.js` |

## Global Object: `window.HitPay`

After the script loads, `window.HitPay` is available with the following methods:

### `HitPay.init(paymentUrl, options)`

Initializes the Drop-In UI with a payment request URL.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `paymentUrl` | string | The `url` field from the payment request response |
| `options` | object | Configuration options (see below) |

**Options:**

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `domain` | string | Yes | `'hit-pay.com'` or `'sandbox.hit-pay.com'` |
| `apiDomain` | string | Yes | `'api.hit-pay.com'` or `'api.sandbox.hit-pay.com'` |
| `onSuccess` | function | No | Called on successful payment |
| `onError` | function | No | Called on payment failure |
| `onClose` | function | No | Called when modal is closed by user |

### `HitPay.toggle(options?)`

Opens or closes the Drop-In modal.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `options.open` | boolean | `true` to open, `false` to close |

If called without options, toggles the current state.

## Callback Signatures

### onSuccess

```typescript
function onSuccess(data: {
  payment_request_id: string;
  status: string;           // "completed"
  reference_number?: string; // Your reference from the payment request
}): void;
```

### onError

```typescript
function onError(error: {
  message: string;  // Human-readable error description
}): void;
```

### onClose

```typescript
function onClose(): void;
// Called when user closes the modal without completing payment
// No parameters — check payment status via API if needed
```

## CSS Customization

The Drop-In modal is rendered in an iframe, so direct CSS targeting is limited. Customization is done via the HitPay dashboard:

1. Go to **Settings > Payment Gateway > Checkout Customization**
2. Options available:
   - **Logo**: Upload your business logo
   - **Primary color**: Button and accent color
   - **Background color**: Modal background

## Content Security Policy (CSP)

If your site uses CSP headers, add the following directives:

```
frame-src https://hit-pay.com https://sandbox.hit-pay.com;
script-src https://hit-pay.com https://sandbox.hit-pay.com;
```

## Supported Payment Methods

The Drop-In UI automatically displays all payment methods enabled on your HitPay account. The customer selects their preferred method within the modal.

**Supported in Drop-In:**
- Credit/Debit Cards (Visa, Mastercard, Amex)
- PayNow
- GrabPay
- ShopeePay
- FPX
- Touch 'n Go
- All other enabled QR methods

**NOT supported in Drop-In:**
- Apple Pay (use redirect flow instead)

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Invalid payment URL | `onError` called with message |
| Payment request expired | `onError` called with message |
| Network error | `onError` called with message |
| 3DS authentication failed | `onError` called with message |
| User closes modal | `onClose` called (not an error) |
| Script not loaded | `window.HitPay` is undefined — check before calling |

## Integration Checklist

- [ ] HitPay.js script loaded before calling `init()`
- [ ] Payment request created server-side (never expose API key to client)
- [ ] `domain` and `apiDomain` match your environment (sandbox vs production)
- [ ] `onSuccess` redirects or updates UI (but does NOT fulfill order)
- [ ] Webhook handler verifies signature and fulfills order
- [ ] CSP headers updated if applicable
- [ ] Tested with sandbox API keys and test cards
