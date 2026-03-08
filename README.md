# HitPay Plugin for Claude Code

Zero-config SEA payment integration for Claude Code. Gives Claude deep knowledge of HitPay's payment gateway plus live API access — so when developers ask for payment integration, Claude scaffolds production-ready code with the right payment methods, webhook handling, and checkout flows.

## What's Inside

| Layer | What It Does | Files |
|-------|-------------|-------|
| **MCP Server** | Live API access — create payments, check statuses, query account | `.mcp.json` |
| **Skills** | Teach Claude correct code patterns for HitPay integration | `skills/` (5 skills) |
| **Commands** | Developer actions — scaffold, lookup, test, QR checkout | `commands/` (4 commands) |

## Installation

### Via Marketplace (recommended)

```
/plugin marketplace add hit-pay/claude-code-plugin
/plugin install hitpay@hitpay-plugins
```

### Local Development

```bash
git clone https://github.com/hit-pay/claude-code-plugin.git hitpay-plugin
claude --plugin-dir ./hitpay-plugin
```

## Setup

Set your HitPay API credentials:

```bash
export HITPAY_API_KEY=your_api_key_here
export HITPAY_SALT=your_webhook_salt_here
```

Get these from your [HitPay Dashboard](https://dashboard.sandbox.hit-pay.com) under Settings > Payment Gateway > API Keys.

## Skills

Skills are auto-invoked when Claude detects relevant context in your conversation.

| Skill | Triggers On | What It Teaches |
|-------|------------|-----------------|
| **payment-integration** | "add HitPay", "payment checkout", "accept payments" | API routes, redirect/QR/selector flows, auth headers, test cards |
| **webhook-handler** | "webhook signature", "payment notification", "verify webhook" | HMAC-SHA256 verification, event types, IP allowlisting, idempotency |
| **payment-methods** | "methods in Malaysia", "which payment methods", "supported methods" | Country-to-method mapping, API values, currencies, unavailable methods |
| **drop-in-ui** | "embed payment form", "HitPay.js", "checkout popup" | HitPay.js script, init options, Next.js integration, event handling |
| **qr-checkout** | "QR checkout page", "QR payment page", "show QR code for PayNow" | Pre-rendered QR images, local HTML page, offline-capable, no CDN |

## Commands

| Command | Description | Example |
|---------|-------------|---------|
| `/hitpay:init` | Scaffold a complete integration | `/hitpay:init nextjs sg,my` |
| `/hitpay:methods` | Look up payment methods by country | `/hitpay:methods ph` |
| `/hitpay:webhook-test` | Generate test webhook payload + curl | `/hitpay:webhook-test charge.created` |
| `/hitpay:qr-checkout` | Generate QR payment page with scannable codes | `/hitpay:qr-checkout 100 sgd paynow,qris` |

## MCP Tools

When `HITPAY_API_KEY` is set, Claude can use these tools during development:

### Read Operations
- `get_account_status` — Check KYC and enabled providers
- `get_balances` — Current account balances
- `get_payment_request` — Payment request details
- `list_payment_requests` — List payment requests
- `list_charges` — List charges with filters
- `get_charge_detail` — Charge details
- `list_customers` — List customers
- `get_customer` — Customer details
- `list_invoices` — List invoices
- `list_subscription_plans` — Subscription plans
- `list_static_qrs` — Static QR codes
- `get_supported_qr_methods` — Available QR methods
- `get_account_info` — Account summary

### Create Operations
- `create_payment_request` — Create payment with checkout URL
- `create_customer` — Create customer record
- `create_invoice` — Create invoice
- `create_embedded_qr` — Generate QR for payment collection
- `create_static_qr` — Create permanent QR (POS)
- `create_webhook_event` — Register webhook URL
- `create_recurring_billing` — Set up subscription

## Supported Markets

| Market | Currency | Key Methods |
|--------|----------|-------------|
| Singapore | SGD | PayNow, GrabPay, ShopeePay, Cards |
| Malaysia | MYR | GrabPay, ShopeePay, Touch 'n Go, FPX |
| Philippines | PHP | QR Ph, GCash, ShopeePay |
| Thailand | THB | PromptPay, TrueMoney |
| Vietnam | VND | VietQR, ZaloPay |
| Indonesia | IDR | QRIS |
| India | INR→SGD | UPI |
| Australia | AUD | PayTo |
| Cross-border | Auto FX | 9 borderless QR methods |

## Security

This plugin follows a safe-by-default approach:

- **Included**: Read operations + payment creation (reversible/safe)
- **Excluded from skills**: Refunds, deletions, transfers, cancellations (destructive/financial operations)
- **Webhook verification**: All generated webhook handlers include HMAC-SHA256 signature checks
- **API keys**: Never exposed client-side in generated code

## License

MIT
