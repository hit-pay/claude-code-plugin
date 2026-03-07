---
name: methods
description: Look up available HitPay payment methods for a country or market. Shows API values, currencies, and notes.
user_invocable: true
---

# /hitpay:methods

Look up available HitPay payment methods. Supports `$ARGUMENTS` for quick filtering (e.g., `/hitpay:methods sg`, `/hitpay:methods philippines`).

## Instructions

### 1. Determine Country

If `$ARGUMENTS` is provided, parse the country:
- `sg`, `singapore` → Singapore
- `my`, `malaysia` → Malaysia
- `ph`, `philippines` → Philippines
- `th`, `thailand` → Thailand
- `vn`, `vietnam` → Vietnam
- `id`, `indonesia` → Indonesia
- `in`, `india` → India
- `au`, `australia` → Australia
- `all` → Show all markets

If no argument, ask: "Which market? (sg, my, ph, th, vn, id, in, au, or all)"

### 2. Look Up Methods

Use the **payment-methods** skill tables to find available methods for the requested country.

### 3. Check Live Status (If MCP Connected)

If the `hitpay-api` MCP server is connected, also call `get_account_status` to show which providers are actually enabled on the developer's account. Mark methods as "Enabled" or "Not enabled" accordingly.

If MCP is not connected, show the static reference tables and note: "Connect your HitPay API key to see which methods are enabled on your account."

### 4. Output Formatted Table

Display results as a formatted table:

```
Payment Methods — Singapore (SGD)

| Method | API Value | Type | Notes |
|--------|-----------|------|-------|
| PayNow | paynow_online | QR | Most popular in SG |
| GrabPay | grabpay | QR | Also available in MY |
| ShopeePay | shopee_pay | QR | Also available in MY, PH |
| Cards | card | Redirect | Visa, MC, Amex |
| Apple Pay | apple_pay | Redirect | Via Stripe |
| Google Pay | google_pay | Redirect | Via Stripe |

QR methods: use generate_qr: true or create_embedded_qr
Card methods: use redirect flow (create_payment_request with redirect_url)
```

### 5. Show Unavailable Methods

If the requested country has commonly-asked-for but unavailable methods, show them:

```
Not available in this market:
- DuitNow → Use GrabPay, ShopeePay, or Touch 'n Go instead
```

### 6. Cross-Border Note

If relevant, mention borderless QR:

```
Cross-border: Your SG account can also accept payments from customers in
PH (GCash, QRPH), TH (PromptPay), ID (QRIS), MY (Touch 'n Go) via
borderless QR. Use /hitpay:methods borderless for details.
```
