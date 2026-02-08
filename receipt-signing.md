---
layout: default
title: Receipt Signing
description: Authenticate receipt callbacks with HMAC-SHA256 to prove they are genuine
permalink: /receipt-signing/
---

# Receipt Signing (HMAC-SHA256)

**The problem:** When your app sends a receipt callback to HookTunnel, how does HookTunnel know it is really from your app and not from an attacker posting fake receipts? If someone can mark unpaid users as "applied confirmed," the entire outcome system is worthless.

Receipt signing solves this. Every async receipt callback must include an HMAC-SHA256 signature computed with your hook's signing secret. HookTunnel verifies the signature before accepting the receipt.

---

## How Receipt Signing Works

1. Each hook has a unique **receipt signing secret** (a cryptographically random key)
2. When your app sends a receipt, it computes an HMAC-SHA256 hash of the request body using the secret
3. Your app includes the hash in the `X-HookTunnel-Signature` header
4. HookTunnel recomputes the hash and compares it
5. If they match, the receipt is accepted. If not, it is rejected with `RCPT_AUTH_INVALID`.

```
Your App                           HookTunnel
   |                                  |
   |  1. Compute HMAC-SHA256          |
   |     of request body              |
   |     using signing secret         |
   |                                  |
   |  2. POST /api/v1/receipts        |
   |     X-HookTunnel-Signature:      |
   |     sha256=<computed-hmac>       |
   | -------------------------------->|
   |                                  |  3. Recompute HMAC
   |                                  |     using stored secret
   |                                  |
   |                                  |  4. Compare signatures
   |                                  |
   |  5. 201 Created (match)          |
   |  or 401 RCPT_AUTH_INVALID        |
   |<---------------------------------|
```

---

## Getting Your Signing Secret

### From the Dashboard

1. Go to your hook's detail page
2. Click **Settings** or the gear icon
3. Find the **Receipt Signing Secret** section
4. Click **Reveal Secret** to show the key

The secret looks like a long random string. Store it securely -- treat it like a password.

### From the API

```bash
curl https://api.hooktunnel.com/api/v1/hooks/<hookId>/receipt-secret \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:

```json
{
  "hook_id": "abc123def456",
  "receipt_signing_secret": "whsec_a1b2c3d4e5f6...",
  "created_at": "2026-02-08T12:00:00Z"
}
```

---

## Signing a Receipt Callback

### Node.js

```javascript
const crypto = require('crypto');

const secret = process.env.HOOKTUNNEL_RECEIPT_SECRET;
const body = JSON.stringify({
  event_id: 'evt_abc123',
  status: 'processed',
  schema_version: '1.0',
  ref: 'user_123_pro_upgrade',
  timestamp: new Date().toISOString(),
});

const signature = crypto
  .createHmac('sha256', secret)
  .update(body)
  .digest('hex');

const response = await fetch('https://api.hooktunnel.com/api/v1/receipts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-HookTunnel-Signature': `sha256=${signature}`,
  },
  body,
});

console.log(response.status); // 201 if accepted
```

### Python

```python
import hmac
import hashlib
import json
import requests
import os
from datetime import datetime

secret = os.environ['HOOKTUNNEL_RECEIPT_SECRET']
body = json.dumps({
    'event_id': 'evt_abc123',
    'status': 'processed',
    'schema_version': '1.0',
    'ref': 'user_123_pro_upgrade',
    'timestamp': datetime.utcnow().isoformat() + 'Z',
})

signature = hmac.new(
    secret.encode('utf-8'),
    body.encode('utf-8'),
    hashlib.sha256
).hexdigest()

response = requests.post(
    'https://api.hooktunnel.com/api/v1/receipts',
    headers={
        'Content-Type': 'application/json',
        'X-HookTunnel-Signature': f'sha256={signature}',
    },
    data=body,
)

print(response.status_code)  # 201 if accepted
```

### curl

```bash
SECRET="your_signing_secret_here"

BODY='{"event_id":"evt_abc123","status":"processed","schema_version":"1.0","ref":"user_123_pro_upgrade","timestamp":"2026-02-08T12:05:00Z"}'

SIGNATURE=$(echo -n "$BODY" | openssl dgst -sha256 -hmac "$SECRET" | sed 's/^.* //')

curl -X POST https://api.hooktunnel.com/api/v1/receipts \
  -H "Content-Type: application/json" \
  -H "X-HookTunnel-Signature: sha256=$SIGNATURE" \
  -d "$BODY"
```

---

## Key Rotation

Over time, you may need to rotate your signing secret -- for example, if a team member leaves or if you suspect the key was exposed.

### How to Rotate

1. Go to your hook's settings or call the API:

```bash
curl -X POST https://api.hooktunnel.com/api/v1/hooks/<hookId>/receipt-secret/rotate \
  -H "Authorization: Bearer YOUR_API_KEY"
```

2. HookTunnel generates a new secret
3. The old secret remains valid for a **24-hour grace period**
4. During the grace period, both the old and new secrets are accepted
5. After 24 hours, only the new secret works

### Grace Period

The 24-hour grace period means you do not need to deploy your secret update simultaneously across all services. You have a full day to update your application with the new secret.

```
Time 0:00  - Rotate secret
             Old secret: valid
             New secret: valid

Time 0:01  - Update your app with new secret
to 23:59     Old secret: still valid (grace period)
             New secret: valid

Time 24:00 - Grace period ends
             Old secret: INVALID
             New secret: valid (only this one works)
```

---

## Troubleshooting

### RCPT_AUTH_INVALID

The signature you sent does not match what HookTunnel computed. Common causes:

1. **Wrong secret** -- Make sure you are using the correct signing secret for this hook. Each hook has its own secret.

2. **Body mismatch** -- The HMAC is computed over the exact request body bytes. If your HTTP library adds whitespace, reorders JSON keys, or modifies the body in any way, the signature will not match. Compute the HMAC over the exact string you send.

3. **Encoding issues** -- Make sure both the secret and body are encoded as UTF-8 before computing the HMAC.

4. **Expired secret** -- If you rotated the secret more than 24 hours ago, the old secret is no longer valid. Use the new one.

### RCPT_AUTH_MISSING

You did not include the `X-HookTunnel-Signature` header. Every async receipt callback must be signed. Add the header:

```
X-HookTunnel-Signature: sha256=<computed-hmac>
```

### Testing Your Signature

To verify your signing implementation is correct:

1. Use a known test payload and secret
2. Compute the HMAC locally
3. Compare it with the expected output

```javascript
// Test with known values
const secret = 'test_secret_123';
const body = '{"event_id":"test","status":"processed","schema_version":"1.0"}';

const expected = crypto
  .createHmac('sha256', secret)
  .update(body)
  .digest('hex');

console.log('Expected signature:', expected);
// Use this to verify your implementation produces the same result
```

---

## Security Best Practices

1. **Store secrets securely** -- Use environment variables or a secrets manager. Never commit signing secrets to version control.

2. **Rotate periodically** -- Even without a security incident, rotate your signing secret on a regular schedule (e.g., quarterly).

3. **Validate in your app too** -- If you receive receipt confirmation webhooks from HookTunnel, validate their signatures as well.

4. **Monitor for auth failures** -- If you see unexpected `RCPT_AUTH_INVALID` errors, investigate. It could be a misconfiguration or an attack attempt.

---

## What's Next?

- [Outcome Receipts](/outcome-receipts/) -- Understand the full receipt system
- [Webhook Replay](/replay/) -- Re-send events that were not applied
- [API Reference](/api-reference/) -- Full API details for receipt endpoints
