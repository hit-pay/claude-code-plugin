---
name: qr-checkout
description: Generate a QR payment checkout page with pre-rendered QR codes. Creates a self-contained HTML file and opens it in the browser.
user_invocable: true
---

# /hitpay:qr-checkout

Generate a branded QR payment checkout page. Creates a local HTML file with pre-rendered QR images (no CDN dependencies) and opens it in the browser.

**Usage:** `/hitpay:qr-checkout <amount> <currency> <methods>`

**Examples:**
- `/hitpay:qr-checkout 100 sgd paynow`
- `/hitpay:qr-checkout 50 sgd paynow,qris,promptpay`
- `/hitpay:qr-checkout 500 php qrph,gcash`

## Instructions

### 1. Parse Arguments

Parse `$ARGUMENTS` as: `<amount> <currency> <methods>`

- `amount` — numeric value (e.g., `100`, `50.00`)
- `currency` — currency code (e.g., `sgd`, `php`, `thb`, `idr`, `myr`)
- `methods` — comma-separated payment method names (case-insensitive)

If arguments are missing or incomplete, ask the user:
- No amount → "What amount should I create the QR for?"
- No currency → "Which currency? (SGD, PHP, MYR, THB, IDR)"
- No methods → "Which payment methods? (paynow, qrph, gcash, qris, promptpay, etc.)"

### 2. Resolve Method Names

Map user-friendly names to API values:

| Name | API Value | Default Currency |
|------|-----------|-----------------|
| paynow | `paynow_online` | SGD |
| qrph, instapay | `qrph_netbank` | PHP |
| gcash | `gcash_qr` | PHP |
| promptpay | `opn_prompt_pay` | THB |
| truemoney | `opn_true_money_qr` | THB |
| qris | `ifpay_qris` | IDR |
| upi | `upi_qr` | SGD |
| shopeepay | `shopee_pay` | SGD |
| grabpay | `grabpay_direct` | SGD |
| touchngo, tng | `touch_n_go` | MYR |

If a method's default currency differs from the specified currency, this is a **borderless (cross-border)** payment. Set currency to the user-specified value (merchant's currency).

### 3. Call MCP Tools

For each resolved method, call `create_embedded_qr`:

```
Tool: create_embedded_qr
Parameters:
  amount: "<amount>"
  currency: "<CURRENCY>"
  payment_methods: ["<api_value>"]
```

Make calls in **parallel** if multiple methods are requested.

### 4. Pre-Render QR Images

For each response, convert the QR payload to a base64 data URI:

**Detect format:**
- `qr_code` starts with `iVBOR` → base64 PNG, prefix with `data:image/png;base64,`
- Anything else → text payload, render with Python:

```bash
pip3 install qrcode[pil] 2>/dev/null
python3 -c "
import qrcode, io, base64
qr = qrcode.make('QR_PAYLOAD_HERE', error_correction=qrcode.constants.ERROR_CORRECT_M)
buf = io.BytesIO()
qr.save(buf, format='PNG')
print('data:image/png;base64,' + base64.b64encode(buf.getvalue()).decode())
"
```

### 5. Assemble HTML

Use the template from the **qr-checkout** skill (`references/html-template.md`):

1. Replace all `{{PLACEHOLDER}}` markers with actual values
2. Duplicate the QR CARD block for each method
3. Set grid class: `single` (1), `cols-2` (2), `cols-3` (3+)
4. Include borderless FX section if `borderless_fx` is present
5. Remove borderless section for domestic payments

Save to `/tmp/hitpay-qr-checkout.html` and open:

```bash
open /tmp/hitpay-qr-checkout.html
```

### 6. Output Summary

```
QR Checkout Page Created!

Methods: PayNow, QRIS, PromptPay
Amount: SGD 100.00
File: /tmp/hitpay-qr-checkout.html

Payment IDs:
- PayNow: abc-123
- QRIS: def-456
- PromptPay: ghi-789

The page is open in your browser. All QR codes are scannable.
```
