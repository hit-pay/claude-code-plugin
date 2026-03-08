---
name: qr-checkout
description: Generate a local QR payment checkout page using HitPay MCP tools. Use when user says "create a QR checkout page", "QR payment page", "show QR code for PayNow", "QR checkout for QRIS", "build a QR payment page", "QR page for [country] customer", or "embedded QR checkout page".
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay QR Checkout — Local Payment Page Generator

Generate a self-contained HTML file that displays a branded QR payment page. Orchestrates `create_embedded_qr` → pre-renders QR images with Python → assembles HTML → opens in browser.

> **Key difference from agent-skills version:** This skill generates local HTML files with pre-rendered QR images (no CDN dependencies). Works offline, on `file://` protocol, and via email.

> **Collecting QR payments without a visual page?** Use the **payment-methods** skill instead.
> **Writing backend integration code?** Use the **payment-integration** skill instead.

## When to Apply

- User wants a **visual QR payment page** as a local HTML file
- User says "create a QR checkout page", "QR payment page", "show a QR to collect payment"
- User mentions a specific payment method + amount and wants a rendered page

## Step 1: Parse Natural Language

Map the user's request to `create_embedded_qr` parameters:

### Payment Method Mapping

| User Says | `payment_methods` | Default Currency |
|-----------|-------------------|-----------------|
| PayNow | `paynow_online` | SGD |
| QRPH / InstaPay | `qrph_netbank` | PHP |
| GCash | `gcash_qr` | PHP |
| PromptPay | `opn_prompt_pay` | THB |
| TrueMoney | `opn_true_money_qr` | THB |
| QRIS | `ifpay_qris` | IDR |
| UPI | `upi_qr` | SGD |
| ShopeePay | `shopee_pay` | SGD |
| GrabPay | `grabpay_direct` | SGD |
| Touch 'n Go / TNG | `touch_n_go` | MYR |

### Customer Country Mapping

| User Says | Recommended Method | Code |
|-----------|--------------------|------|
| Filipino / Philippine customer | QR Ph (InstaPay) | `qrph_netbank` |
| Thai customer | PromptPay | `opn_prompt_pay` |
| Indonesian customer | QRIS | `ifpay_qris` |
| Malaysian customer | Touch 'n Go | `touch_n_go` |
| Singapore customer | PayNow | `paynow_online` |

### Display Name Lookup

| API Code | Display Name |
|----------|-------------|
| `paynow_online` | PayNow |
| `qrph_netbank` | QRPH (InstaPay) |
| `gcash_qr` | GCash |
| `opn_prompt_pay` | PromptPay |
| `opn_true_money_qr` | TrueMoney |
| `ifpay_qris` | QRIS |
| `doku_qris` | QRIS (DOKU) |
| `upi_qr` | UPI |
| `shopee_pay` | ShopeePay |
| `grabpay_direct` | GrabPay |
| `touch_n_go` | Touch 'n Go |

## Step 2: Detect Borderless (Cross-Border)

If the customer's country differs from the merchant's currency country:

1. Set `currency` to the **merchant's** currency (e.g., `SGD` for a Singapore merchant)
2. Use the **borderless method code** from the table above (e.g., `gcash_qr` not `gcash`)
3. The response will include `borderless_fx` with FX conversion details

**Example:** Singapore merchant + Filipino customer → `currency: "SGD"`, `payment_methods: ["qrph_netbank"]`

## Step 3: Call MCP Tool

For each payment method, call `create_embedded_qr`:

```
Tool: create_embedded_qr
Parameters:
  amount: "<amount>"
  currency: "<resolved currency>"
  payment_methods: ["<resolved method>"]
  purpose: "<optional description>"
  reference_number: "<optional reference>"
```

If the user requests multiple methods, make **separate** `create_embedded_qr` calls for each method (the tool accepts only one method at a time).

### Expected Response Shape

```json
{
  "id": "abc-123",
  "amount": "SGD 100.00",
  "status": "pending",
  "payment_methods": ["qrph_netbank"],
  "checkout_url": "https://securecheckout.hit-pay.com/...",
  "qr_code_data": {
    "qr_code": "<raw QR payload string or base64 PNG>",
    "qr_code_expiry": "2025-01-01T12:15:00Z"
  },
  "borderless_fx": {
    "customer_pays": "PHP 4,557.00",
    "merchant_receives": "SGD 100.00",
    "fx_rate": "45.57",
    "display_rate": "1 SGD = 45.57 PHP",
    "fee_note": "1.5% cross-border processing fee applies"
  }
}
```

## Step 4: Pre-Render QR Images with Python

**This is the critical step.** Convert QR payloads to base64 data URIs so the HTML works on `file://` without any CDN.

### QR Format Detection

Check the `qr_code_data.qr_code` value:

- **Starts with `iVBOR`** → Already a base64 PNG. Prefix with `data:image/png;base64,` to create the data URI.
- **Anything else** → Raw text payload (e.g., PayNow EMV string, QRIS TLV). Must be rendered to a QR image.

### Rendering Text Payloads

Use Python's `qrcode` library to convert text payloads to base64 PNG data URIs:

```bash
# First, ensure qrcode is installed
pip3 install qrcode[pil] 2>/dev/null

# Then render the QR payload to a base64 data URI
python3 -c "
import qrcode, io, base64
qr = qrcode.make('PASTE_QR_PAYLOAD_HERE', error_correction=qrcode.constants.ERROR_CORRECT_M)
buf = io.BytesIO()
qr.save(buf, format='PNG')
print('data:image/png;base64,' + base64.b64encode(buf.getvalue()).decode())
"
```

**Important:** The QR payload may contain special characters. Pass it as a Python string literal (use raw strings or escape properly). For very long payloads, write to a temp file first:

```bash
# For long/complex payloads
echo 'PAYLOAD_HERE' > /tmp/qr_payload.txt
python3 -c "
import qrcode, io, base64
with open('/tmp/qr_payload.txt') as f:
    payload = f.read().strip()
qr = qrcode.make(payload, error_correction=qrcode.constants.ERROR_CORRECT_M)
buf = io.BytesIO()
qr.save(buf, format='PNG')
print('data:image/png;base64,' + base64.b64encode(buf.getvalue()).decode())
"
```

### Handling Base64 PNG Payloads

If the `qr_code` starts with `iVBOR`, it's already a base64 PNG:

```
data:image/png;base64,iVBOR...
```

Just prepend `data:image/png;base64,` — no Python rendering needed.

## Step 5: Assemble HTML Page

Read the template from `references/html-template.md` and inject the data:

1. Replace all `{{PLACEHOLDER}}` markers with actual values
2. For **single method**: use the single-card layout
3. For **multiple methods**: use the grid layout (the template handles both)
4. Set borderless section visibility based on whether `borderless_fx` is present

Save the file and open it:

```bash
# Save to a descriptive filename
open /tmp/hitpay-qr-checkout-{method}-{amount}.html
```

## Error Handling

| Situation | Action |
|-----------|--------|
| Missing amount | Ask user: "What amount should I create the QR for?" |
| Missing payment method | Ask user: "Which payment method? (PayNow, QRPH, GCash, etc.)" |
| Ambiguous method | Default to PayNow (SGD) or ask if multi-market |
| `pip3 install qrcode` fails | Fall back: use the `checkout_url` and tell user to open in browser |
| Provider not enabled | Tell user to enable in HitPay dashboard |
| API error | Surface the error message from the MCP response |
| `qr_code_data` is null | Use `checkout_url` — generate an HTML page with just the checkout link |

## Brand Colors

| Token | Hex | Usage |
|-------|-----|-------|
| Deep Blue | `#002771` | Dark gradient background |
| Logo Blue | `#0E2859` | Card background |
| Action Blue | `#2465DE` | Accents, links, badges |
| White | `#FFFFFF` | QR container, text on dark |
| Success Green | `#4DAB80` | Merchant receives amount |
| Danger | `#E5484D` | Expired timer |

## Skill Boundaries

- **This skill:** Orchestrates MCP call → pre-render QR → local HTML file (no CDN, works offline)
- **payment-methods:** Country-to-method lookup tables, API values
- **payment-integration:** Backend integration code — API routes, webhook handlers
- **drop-in-ui:** HitPay.js embedded checkout (requires network)
