---
name: webhook-test
description: Generate a realistic HitPay webhook test payload with HMAC signature and a ready-to-use curl command.
user_invocable: true
---

# /hitpay:webhook-test

Generate a realistic webhook test payload with a valid HMAC-SHA256 signature and a ready-to-use curl command. Supports `$ARGUMENTS` for event type (e.g., `/hitpay:webhook-test charge.created`).

## Instructions

### 1. Determine Event Type

If `$ARGUMENTS` is provided, use it as the event type:
- `completed` or `payment_request.completed` → Payment completed webhook
- `failed` or `payment_request.failed` → Payment failed webhook
- `charge.created` → Charge created webhook
- `charge.updated` → Charge updated webhook

Default: `payment_request.completed`

### 2. Determine Webhook Salt

Check for `HITPAY_SALT` in the environment or `.env.local` / `.env` files. If not found, use a placeholder: `test_salt_replace_me`.

### 3. Generate Payload

Create a realistic JSON payload based on the event type. Use the **webhook-handler** skill payload examples as the template.

For `payment_request.completed`:

```json
{
  "id": "test_pr_<random_uuid>",
  "amount": "100.00",
  "currency": "sgd",
  "status": "completed",
  "reference_number": "TEST-ORDER-001",
  "email": "test@example.com",
  "name": "Test Customer",
  "payment_type": "paynow_online",
  "payments": [
    {
      "id": "test_pay_<random_uuid>",
      "amount": "100.00",
      "currency": "sgd",
      "status": "succeeded"
    }
  ],
  "created_at": "<current_iso_timestamp>",
  "updated_at": "<current_iso_timestamp>"
}
```

For `payment_request.failed`:

```json
{
  "id": "test_pr_<random_uuid>",
  "amount": "100.00",
  "currency": "sgd",
  "status": "failed",
  "reference_number": "TEST-ORDER-002",
  "failure_reason": "card_declined",
  "created_at": "<current_iso_timestamp>",
  "updated_at": "<current_iso_timestamp>"
}
```

### 4. Compute HMAC Signature

Compute the HMAC-SHA256 signature of the JSON payload using the webhook salt:

```bash
SIGNATURE=$(echo -n '$PAYLOAD' | openssl dgst -sha256 -hmac '$SALT' | awk '{print $2}')
```

### 5. Output Curl Command

Generate a ready-to-use curl command targeting `localhost:3000/api/webhooks/hitpay` (or detect the webhook route from the project):

```bash
curl -X POST http://localhost:3000/api/webhooks/hitpay \
  -H "Content-Type: application/json" \
  -H "Hitpay-Signature: <computed_signature>" \
  -H "Hitpay-Event-Type: completed" \
  -H "Hitpay-Event-Object: payment_request" \
  -H "User-Agent: HitPay v2.0" \
  -d '<json_payload>'
```

### 6. Output Summary

```
Webhook Test Payload Generated

Event: payment_request.completed
Salt used: <salt_source>
Signature: <hex_signature>

Copy and run the curl command above with your dev server running.
Expected response: {"received": true}

If you get 401, check that HITPAY_SALT in your .env matches the salt used above.
```

### 7. Optional: Run the Test

Ask: "Want me to run this curl command now? (Make sure your dev server is running on localhost:3000)"

If yes, execute the curl command and report the response.
