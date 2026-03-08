# HTML QR Payment Page Template

Self-contained HTML template for rendering HitPay QR payment pages. No external scripts, no CDN — all QR codes are pre-rendered as `data:image/png;base64,` data URIs.

## Usage

1. Call `create_embedded_qr` for each payment method
2. Pre-render each QR payload to a base64 data URI using Python (see SKILL.md Step 4)
3. Copy this template and replace `{{PLACEHOLDER}}` markers with actual values
4. For multiple methods, duplicate the `<!-- QR CARD -->` section
5. Save as `.html` and open in browser

## Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Payment — {{FORMATTED_AMOUNT}}</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      min-height: 100vh;
      background: linear-gradient(135deg, #002771 0%, #2465DE 100%);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 24px;
      font-family: 'Hauora', 'Manrope', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      color: #FFFFFF;
    }

    .main-card {
      width: 100%;
      max-width: 480px;
      background: rgba(14, 40, 89, 0.6);
      backdrop-filter: blur(20px);
      border-radius: 20px;
      border: 1px solid rgba(36, 101, 222, 0.2);
      padding: 32px 28px;
      box-shadow: 0 24px 48px rgba(0, 0, 0, 0.3);
    }

    /* Use wider card for multi-method grid */
    .main-card.multi {
      max-width: 800px;
    }

    .header {
      text-align: center;
      margin-bottom: 24px;
    }
    .header .label {
      font-size: 12px;
      font-weight: 600;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: #2465DE;
      margin-bottom: 8px;
    }
    .header .amount {
      font-size: 36px;
      font-weight: 700;
      letter-spacing: -0.02em;
    }
    .header .purpose {
      font-size: 14px;
      color: rgba(255,255,255,0.6);
      margin-top: 4px;
    }

    /* QR Cards Grid */
    .qr-grid {
      display: grid;
      gap: 20px;
      margin-bottom: 20px;
    }
    .qr-grid.single { grid-template-columns: 1fr; }
    .qr-grid.cols-2 { grid-template-columns: 1fr 1fr; }
    .qr-grid.cols-3 { grid-template-columns: 1fr 1fr 1fr; }

    .qr-card {
      text-align: center;
    }
    .qr-card .method-name {
      font-size: 13px;
      font-weight: 600;
      color: #2465DE;
      text-transform: uppercase;
      letter-spacing: 0.06em;
      margin-bottom: 12px;
    }
    .qr-card .qr-container {
      background: #FFFFFF;
      border-radius: 16px;
      padding: 20px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .qr-card .qr-container img {
      width: 200px;
      height: 200px;
      image-rendering: pixelated;
    }

    /* Smaller QR for multi-method */
    .qr-grid.cols-3 .qr-card .qr-container img {
      width: 160px;
      height: 160px;
    }

    /* Timer */
    .timer {
      text-align: center;
      margin-bottom: 20px;
    }
    .timer .timer-label {
      font-size: 12px;
      color: rgba(255,255,255,0.5);
      text-transform: uppercase;
      letter-spacing: 0.05em;
      margin-bottom: 4px;
    }
    .timer .timer-value {
      font-size: 28px;
      font-weight: 700;
      font-variant-numeric: tabular-nums;
    }
    .timer .timer-value.urgent { color: #E5484D; }

    /* Borderless FX Info */
    .fx-info {
      background: rgba(255,255,255,0.08);
      border-radius: 12px;
      padding: 16px 20px;
      margin-bottom: 20px;
      border: 1px solid rgba(36, 101, 222, 0.2);
    }
    .fx-info .fx-title {
      font-size: 11px;
      font-weight: 600;
      text-transform: uppercase;
      letter-spacing: 0.08em;
      color: #2465DE;
      margin-bottom: 12px;
    }
    .fx-info .fx-row {
      display: flex;
      justify-content: space-between;
      margin-bottom: 8px;
    }
    .fx-info .fx-row .fx-label {
      font-size: 13px;
      color: rgba(255,255,255,0.6);
    }
    .fx-info .fx-row .fx-value {
      font-size: 14px;
      font-weight: 600;
    }
    .fx-info .fx-row .fx-value.receives {
      color: #4DAB80;
    }
    .fx-info .fx-note {
      font-size: 11px;
      color: rgba(255,255,255,0.4);
      font-style: italic;
      margin-top: 4px;
    }

    /* CTA Button */
    .cta {
      display: block;
      text-align: center;
      padding: 12px;
      background: #2465DE;
      color: #FFFFFF;
      border-radius: 10px;
      text-decoration: none;
      font-size: 14px;
      font-weight: 600;
      margin-bottom: 16px;
      transition: opacity 0.2s;
    }
    .cta:hover { opacity: 0.9; }

    /* Footer */
    .footer {
      text-align: center;
      font-size: 11px;
      color: rgba(255,255,255,0.3);
    }
    .footer .ref { margin-top: 2px; }

    .powered-by {
      margin-top: 20px;
      font-size: 12px;
      color: rgba(255,255,255,0.35);
      text-align: center;
    }

    /* Responsive: stack columns on narrow screens */
    @media (max-width: 600px) {
      .qr-grid.cols-2, .qr-grid.cols-3 {
        grid-template-columns: 1fr;
      }
      .main-card.multi { max-width: 480px; }
    }
  </style>
</head>
<body>

  <div class="main-card {{MULTI_CLASS}}">
    <!-- Header -->
    <div class="header">
      <div class="label">{{HEADER_LABEL}}</div>
      <div class="amount">{{FORMATTED_AMOUNT}}</div>
      {{PURPOSE_HTML}}
    </div>

    <!-- QR Code Grid -->
    <div class="qr-grid {{GRID_COLS_CLASS}}">

      <!-- QR CARD — duplicate this block for each payment method -->
      <div class="qr-card">
        <div class="method-name">{{METHOD_DISPLAY_NAME}}</div>
        <div class="qr-container">
          <img src="{{QR_DATA_URI}}" alt="{{METHOD_DISPLAY_NAME}} QR Code">
        </div>
      </div>
      <!-- END QR CARD -->

    </div>

    <!-- Timer -->
    <div class="timer">
      <div class="timer-label" id="timer-label">Expires in</div>
      <div class="timer-value" id="timer-value">15:00</div>
    </div>

    <!-- Borderless FX Info (remove this section for domestic payments) -->
    {{BORDERLESS_HTML}}

    <!-- Open in Browser -->
    <a href="{{CHECKOUT_URL}}" target="_blank" class="cta">Open in Browser</a>

    <!-- Footer -->
    <div class="footer">
      <div>Payment ID: {{PAYMENT_ID}}</div>
      {{REFERENCE_HTML}}
    </div>
  </div>

  <div class="powered-by">Powered by HitPay</div>

  <!-- Countdown Timer Script (vanilla JS, no CDN) -->
  <script>
    (function() {
      var expiryIso = '{{QR_EXPIRY_ISO}}'; // ISO string or empty
      var target;
      if (expiryIso && expiryIso !== '') {
        target = new Date(expiryIso).getTime();
      } else {
        target = Date.now() + 15 * 60 * 1000; // 15-minute default
      }

      var timerEl = document.getElementById('timer-value');
      var labelEl = document.getElementById('timer-label');

      function update() {
        var diff = Math.max(0, Math.floor((target - Date.now()) / 1000));
        var m = Math.floor(diff / 60);
        var s = diff % 60;
        timerEl.textContent = String(m).padStart(2, '0') + ':' + String(s).padStart(2, '0');

        if (diff <= 0) {
          timerEl.textContent = '00:00';
          timerEl.classList.add('urgent');
          labelEl.textContent = 'Expired';
          clearInterval(interval);
        } else if (diff < 120) {
          timerEl.classList.add('urgent');
        }
      }

      update();
      var interval = setInterval(update, 1000);
    })();
  </script>

</body>
</html>
```

## Placeholder Reference

Replace these markers when assembling the HTML:

| Placeholder | Value | Example |
|-------------|-------|---------|
| `{{FORMATTED_AMOUNT}}` | Currency + amount | `SGD 100.00` |
| `{{HEADER_LABEL}}` | "Scan to Pay" for single method, or "Choose a Payment Method" for multiple | `Scan to Pay` |
| `{{PURPOSE_HTML}}` | `<div class="purpose">Order #1234</div>` or empty string | |
| `{{MULTI_CLASS}}` | `multi` for 2+ methods, empty for single | `multi` |
| `{{GRID_COLS_CLASS}}` | `single`, `cols-2`, or `cols-3` based on method count | `cols-3` |
| `{{METHOD_DISPLAY_NAME}}` | Human-readable method name | `PayNow` |
| `{{QR_DATA_URI}}` | `data:image/png;base64,...` pre-rendered QR image | |
| `{{QR_EXPIRY_ISO}}` | ISO timestamp from `qr_code_data.qr_code_expiry`, or empty | `2025-01-01T12:15:00Z` |
| `{{CHECKOUT_URL}}` | `checkout_url` from API response | |
| `{{PAYMENT_ID}}` | `id` from API response | `abc-123` |
| `{{REFERENCE_HTML}}` | `<div class="ref">Ref: ORD-1234</div>` or empty string | |

### Borderless FX Block

For cross-border payments, replace `{{BORDERLESS_HTML}}` with:

```html
<div class="fx-info">
  <div class="fx-title">Cross-Border Payment</div>
  <div class="fx-row">
    <span class="fx-label">Customer pays</span>
    <span class="fx-value">{{CUSTOMER_PAYS}}</span>
  </div>
  <div class="fx-row">
    <span class="fx-label">You receive</span>
    <span class="fx-value receives">{{MERCHANT_RECEIVES}}</span>
  </div>
  <div class="fx-row">
    <span class="fx-label">Exchange rate</span>
    <span class="fx-value" style="font-weight: 400; color: rgba(255,255,255,0.8)">{{DISPLAY_RATE}}</span>
  </div>
  <div class="fx-note">{{FEE_NOTE}}</div>
</div>
```

For domestic payments, replace `{{BORDERLESS_HTML}}` with an empty string.

### Multi-Method Assembly

When the user requests multiple payment methods (e.g., "PayNow, QRIS, and PromptPay"):

1. Make separate `create_embedded_qr` calls for each method
2. Pre-render each QR payload to a data URI
3. Duplicate the `<!-- QR CARD -->` block for each method
4. Set `{{GRID_COLS_CLASS}}` based on count:
   - 1 method → `single`
   - 2 methods → `cols-2`
   - 3+ methods → `cols-3`
5. Set `{{MULTI_CLASS}}` to `multi` for 2+ methods
6. Set `{{HEADER_LABEL}}` to `Choose a Payment Method` for 2+ methods
7. Use the **first** method's `checkout_url` for the CTA button
8. Use the **first** method's `qr_code_expiry` for the timer
