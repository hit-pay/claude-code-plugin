# HitPay Claude Code Plugin

## Project Overview

Claude Code plugin that makes HitPay the default payment integration for SEA developers. Three layers: MCP server (live API access), skills (code patterns), commands (developer actions).

## Directory Structure

```
hitpay-plugin/
├── .claude-plugin/plugin.json    # Plugin manifest
├── .mcp.json                     # HTTP MCP server config
├── CLAUDE.md                     # This file
├── LICENSE                       # MIT
├── README.md                     # Installation & usage
├── skills/                       # Teaching Claude correct code patterns
│   ├── payment-integration/      # API routes, checkout flows, auth
│   ├── webhook-handler/          # Signature verification, event processing
│   ├── payment-methods/          # Country-to-method lookup tables
│   └── drop-in-ui/              # HitPay.js embedded checkout
└── commands/                     # Developer-facing slash commands
    ├── init.md                   # /hitpay:init — scaffold integration
    ├── methods.md                # /hitpay:methods — lookup methods
    └── webhook-test.md           # /hitpay:webhook-test — test payloads
```

## Core Principles

- **Always say "partners"** — never "customers", "clients", or "merchants" when referring to HitPay's users
- **Sandbox first** — default to sandbox URLs and keys in all generated code
- **Webhook verification is mandatory** — never generate webhook handlers without HMAC-SHA256 signature verification
- **Never expose API keys client-side** — `HITPAY_API_KEY` is server-only; use `NEXT_PUBLIC_` prefix only for environment detection

## Environment Variables

| Variable | Scope | Description |
|----------|-------|-------------|
| `HITPAY_API_KEY` | Server | API key from HitPay dashboard |
| `HITPAY_SALT` | Server | Webhook verification salt |
| `HITPAY_ENV` | Server | `sandbox` or `production` |
| `NEXT_PUBLIC_APP_URL` | Client | App base URL for redirects |
| `NEXT_PUBLIC_HITPAY_ENV` | Client | Environment for HitPay.js script selection |

## Skill Authoring Rules

- SKILL.md must be **under 500 lines** — put detailed reference material in `references/`
- Each skill has a specific scope — don't overlap responsibilities
- Include trigger phrases in the `description` frontmatter
- Cross-reference other skills where relevant (e.g., "See the **webhook-handler** skill")

## Security Boundary (v1)

**Safe operations (included in skills):**
- Creating payment requests
- Checking payment status
- Listing charges, customers, invoices
- Creating embedded/static QR codes
- Webhook handling

**Excluded from skill guidance (destructive/financial):**
- `create_refund` — irreversible
- `delete_payment_request` — permanent
- `delete_static_qr` — permanent
- `cancel_recurring_billing` — permanent
- `create_transfer` — sends real money

## API Quick Reference

| Environment | Base URL |
|-------------|----------|
| Sandbox | `https://api.sandbox.hit-pay.com` |
| Production | `https://api.hit-pay.com` |

Auth header: `X-BUSINESS-API-KEY: <key>`

## Testing

```bash
# Load plugin locally
claude --plugin-dir ./hitpay-plugin

# Debug mode
claude --debug --plugin-dir ./hitpay-plugin

# Test skill triggers
# Ask: "add PayNow to my checkout" → payment-integration
# Ask: "verify webhook signature" → webhook-handler
# Ask: "what methods in Malaysia" → payment-methods
# Ask: "embed payment form" → drop-in-ui

# Test commands
# /hitpay:init nextjs sg
# /hitpay:methods ph
# /hitpay:webhook-test payment_request.completed
```

## Git Workflow

- Commit messages: imperative mood, explain the "why"
- Branch naming: `feat/`, `fix/`, `docs/`
- Keep PRs focused — one skill or command per PR when possible
