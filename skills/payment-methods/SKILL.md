---
name: payment-methods
description: Look up HitPay payment methods by country, currency, and provider. Use when user says "which payment methods in Malaysia", "what methods for Singapore", "HitPay payment methods", "payment method lookup", "which QR methods", "supported methods", or "available payment methods".
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay Payment Methods Reference

Lookup reference for payment methods available across Southeast Asia, India, Australia, and cross-border markets. Maps countries to methods, API values, currencies, and settlement behavior.

## When to Apply

- Developer asks which payment methods are available for a specific country
- Need the correct `payment_methods` + `currency` combination for API calls
- Determining whether a method settles in local currency or cross-currency
- Checking if a specific method (e.g., DuitNow, DANA) is available

> **For live account queries**, use the `hitpay-api` MCP tools: `get_account_status` shows which providers are enabled, `get_supported_qr_methods` shows available QR methods.

## Country-to-Method Quick Lookup

| Country | Recommended QR Methods | Currency |
|---------|----------------------|----------|
| **Singapore** | PayNow, GrabPay, ShopeePay | SGD |
| **Malaysia** | GrabPay, ShopeePay, Touch 'n Go, FPX | MYR |
| **Philippines** | QR Ph (InstaPay), ShopeePay, GCash | PHP |
| **Thailand** | PromptPay, TrueMoney | THB |
| **Vietnam** | VietQR, ZaloPay | VND |
| **Indonesia** | QRIS | IDR |
| **India** | UPI | INR (settles SGD) |
| **Australia** | PayTo | AUD |
| **Korea** | KCP methods | KRW |
| **Cross-border (Chinese tourists)** | WeChat Pay, Alipay+ | CNY (settles SGD) |

If the developer doesn't specify a country, ask. The wrong currency will cause the API call to fail.

## Payment Method API Values

These are the exact strings to pass in the `payment_methods` parameter:

| Display Name | API Value | Currency | Notes |
|-------------|-----------|----------|-------|
| PayNow | `paynow_online` | SGD | QR-based |
| GrabPay | `grabpay` | SGD, MYR | QR-based |
| ShopeePay | `shopee_pay` | SGD, MYR, PHP | QR-based |
| Touch 'n Go | `touch_n_go` | MYR | QR-based |
| FPX | `fpx` | MYR | Bank transfer (redirect), not QR |
| PromptPay | `promptpay` | THB | QR-based (domestic) |
| TrueMoney | `truemoney` | THB | QR-based (domestic) |
| VietQR | `vietqr` | VND | QR-based |
| ZaloPay | `zalopay` | VND | QR-based |
| QRIS | `qris` | IDR | QR-based (covers GoPay, OVO, DANA) |
| QR Ph | `qrph` | PHP | InstaPay rail |
| GCash | `gcash` | PHP | QR-based (domestic) |
| UPI | `upi` | SGD | Cross-currency (INR customer, SGD settlement) |
| WeChat Pay | `wechat` | SGD | Cross-currency (CNY customer) |
| Alipay+ | `alipay` | SGD | Cross-currency (CNY customer) |
| PayTo | `payto` | AUD | Bank-initiated payment |
| Cards | `card` | Any | Visa, MC, Amex — redirect checkout |

## Borderless QR Methods (Cross-Border)

For cross-border payments, some methods use different API codes than domestic. **Using the domestic code for a cross-border call will fail.**

| Display Name | Borderless API Value | Country | Notes |
|-------------|---------------------|---------|-------|
| PayNow | `paynow_online` | SG | Same as domestic |
| GCash | `gcash_qr` | PH | Different from domestic `gcash` |
| QR Ph | `qrph_netbank` | PH | Same as domestic |
| PromptPay | `opn_prompt_pay` | TH | Different from domestic `promptpay` |
| TrueMoney | `opn_true_money_qr` | TH | Different from domestic `truemoney` |
| QRIS (iFPay) | `ifpay_qris` | ID | Different from domestic `qris` |
| QRIS (DOKU) | `doku_qris` | ID | Different from domestic `qris` |
| Touch 'n Go | `touch_n_go` | MY | Same as domestic |
| ShopeePay | `shopee_pay` | Multi | Same as domestic |

**Key rule:** For borderless, set `currency` to the **merchant's** currency (e.g., SGD), not the customer's. The response includes `borderless_fx` with conversion details.

## Cross-Currency Settlement

| Method | Customer Pays | Merchant Receives | FX Applied |
|--------|--------------|-------------------|------------|
| WeChat Pay | CNY | SGD | At transaction time |
| Alipay+ | CNY | SGD | At transaction time |
| UPI | INR | SGD | At transaction time |
| Borderless QR | Foreign currency | Merchant's currency | At transaction time |
| All others | Local currency | Same (local) | No FX |

**Rule**: For cross-currency methods, always set `currency` to the **settlement currency** (SGD), not the customer's local currency.

## Known Unavailable Methods

These are commonly requested but **not available** as separate methods on HitPay:

| Requested Method | Country | Alternative |
|-----------------|---------|-------------|
| DuitNow | Malaysia | GrabPay, ShopeePay, Touch 'n Go |
| DANA | Indonesia | QRIS (covers DANA wallets) |
| OVO | Indonesia | QRIS (covers OVO) |
| GoPay | Indonesia | QRIS (covers GoPay) |
| LINE Pay | Thailand | PromptPay, TrueMoney |
| Momo | Vietnam | VietQR, ZaloPay |
| Boost | Malaysia | GrabPay, ShopeePay, Touch 'n Go |
| PayPay | Japan | Not available |

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Payment method not available" | Provider not enabled | Enable in HitPay dashboard, check `get_account_status` |
| "Invalid currency" | Currency doesn't match method | Use the correct currency from the tables above |
| "Amount too low" | Below minimum threshold | Minimum is usually 1.00 in local currency |
| Provider in `pending[]` | KYC not complete | Complete verification in HitPay dashboard |

## Detailed Provider Maps

See `references/provider-currency-map.md` for the full provider-to-method mapping with settlement details for each market.
