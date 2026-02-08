---
layout: default
title: Outcome Receipts
description: Prove your app applied business changes, not just that a webhook was delivered
permalink: /outcome-receipts/
---

# Outcome Receipts: Proving Business Outcomes

**The problem:** Stripe says a customer paid, and your app received the webhook. But did your app actually grant Pro access? Mark the invoice as paid? Provision the account? A `200 OK` response only means "I heard you." It does not mean "I did something about it."

Outcome Receipts let your app confirm to HookTunnel that it applied the business change. This turns HookTunnel from "a webhook forwarder" into "proof that customers got what they paid for."

---

## The Three Statuses That Matter

Every payment-related webhook event in HookTunnel has three independent statuses:

| Status | Source | What It Means |
|--------|--------|---------------|
| **Paid** | Stripe | Stripe reports this payment succeeded |
| **Delivered** | HookTunnel | Your app received the event and responded OK |
| **Applied** | Your App (Receipt) | Your app confirmed it made the change |

The dangerous state is **Delivered but Applied Unknown**: "We sent the event, your app said OK, but we cannot prove the user got what they paid for."

Outcome Receipts exist to close that gap.

---

## How Receipts Work

Your app sends a **receipt** to HookTunnel after it has completed the business logic triggered by a webhook event. The receipt says: "I committed the business change triggered by event X."

```
Stripe sends payment webhook
        |
        v
HookTunnel captures and forwards to your app
        |
        v
Your app processes the payment (grants access, etc.)
        |
        v
Your app sends a receipt back to HookTunnel
        |
        v
HookTunnel marks the event as "Applied Confirmed"
```

If your app does not send a receipt within the configured SLA (default: 5 minutes), HookTunnel marks the event as **Applied Unknown** with an actionable next step.

---

## Three Ways to Send Receipts

### 1. Response Header (Simplest)

Include a receipt header in your webhook handler's HTTP response:

```
X-HookTunnel-Receipt: status=processed
```

This works best when your app processes the webhook synchronously (within the same HTTP request).

**Example (Node.js / Express):**

```javascript
app.post('/webhook', (req, res) => {
  // Process the payment...
  grantProAccess(req.body.data.object.customer);

  // Tell HookTunnel it was applied
  res.set('X-HookTunnel-Receipt', 'status=processed');
  res.status(200).json({ received: true });
});
```

### 2. Response Body (Inline)

Include a `_hooktunnel` object in your JSON response body:

```json
{
  "received": true,
  "_hooktunnel": {
    "receipt": {
      "status": "processed",
      "ref": "user_123_pro_upgrade"
    }
  }
}
```

### 3. Async Callback (Recommended for Production)

Most production systems acknowledge webhooks immediately and process them asynchronously (via a queue or background job). In this case, send a receipt callback after processing:

```bash
curl -X POST https://api.hooktunnel.com/api/v1/receipts \
  -H "Content-Type: application/json" \
  -H "X-HookTunnel-Signature: sha256=<computed-hmac>" \
  -d '{
    "event_id": "evt_abc123",
    "status": "processed",
    "schema_version": "1.0",
    "ref": "user_123_pro_upgrade",
    "timestamp": "2026-02-08T12:05:00Z"
  }'
```

This is the recommended approach because it matches how real production systems work. See [Receipt Signing](/receipt-signing/) for how to authenticate these callbacks.

---

## Receipt Status Values

When sending a receipt, the `status` field must be one of:

| Status | Meaning |
|--------|---------|
| `processed` | Your app successfully applied the business change |
| `failed` | Your app attempted the change but it failed |
| `queued` | Your app has queued the change for later processing |

Do **not** use `success`, `error`, or other values. HookTunnel requires explicit receipt semantics.

---

## Processing Status States

Each event's processing status follows this state machine:

```
not_required ─────────────────────────────> (no receipt needed)
         |
         v
  applied_unknown ──── receipt arrives ───> applied_confirmed
         |                                         |
         └──── receipt says failed ───> applied_failed
```

| State | Meaning |
|-------|---------|
| `not_required` | This event type does not need a receipt |
| `applied_unknown` | Delivered but no receipt received (yet) |
| `applied_confirmed` | Your app confirmed it applied the change |
| `applied_failed` | Your app reported it could not apply the change |

### The Dangerous State

**`applied_unknown`** is the revenue leak zone. It means "we delivered the webhook, but we cannot prove the user got what they paid for." Your goal is to move events out of this state, either by sending receipts or by investigating the gap.

---

## Outcome Status Display

In the dashboard, each event shows three rows:

| Row | Example Value | Source |
|-----|---------------|--------|
| **Paid** | Stripe reports payment succeeded | Stripe event data |
| **Delivered** | Sent to your app, responded 200 | HookTunnel delivery log |
| **Applied** | Your app confirmed (or unknown) | Receipt |

Hover over any status label to see a plain-language tooltip explaining what it means:

- **"Stripe reports this payment succeeded."**
- **"We sent the event to your app and it responded OK."**
- **"Your app confirmed it applied the change."**
- **"Your app hasn't confirmed applying the change. It may have applied it, but we can't prove it."**
- **"Your app reported it could not apply the change."**

Click a status label to highlight the exact trace step with supporting evidence.

---

## Reason Codes

When a receipt is rejected or a status changes, HookTunnel uses standardized reason codes:

| Code | Meaning | What to Do |
|------|---------|------------|
| `RCPT_ACCEPTED_PROCESSED` | Receipt accepted, status set to applied_confirmed | Nothing -- this is the happy path |
| `RCPT_ACCEPTED_FAILED` | Receipt accepted, status set to applied_failed | Investigate why your app could not apply the change |
| `RCPT_AUTH_INVALID` | Signature verification failed | Check your signing secret -- see [Receipt Signing](/receipt-signing/) |
| `RCPT_AUTH_MISSING` | No signature provided | Include the `X-HookTunnel-Signature` header |
| `RCPT_EVENT_NOT_FOUND` | Referenced event_id does not exist | Verify the event_id matches a captured event |
| `RCPT_SCHEMA_INVALID` | Receipt payload does not match expected schema | Check required fields: event_id, status, schema_version |
| `RCPT_DUPLICATE_IGNORED` | Same receipt was already accepted | Safe to ignore -- receipts are idempotent |
| `RCPT_MISSING_SLA` | No receipt received within the SLA window | Your app may have failed silently -- investigate |
| `RCPT_STATUS_REGRESSED` | Attempted to revert from confirmed to unknown | Requires explicit override with audit note |
| `RCPT_OVERRIDE_APPLIED` | Status was overridden with audit trail | Administrative action recorded in audit log |

---

## Key Safety Properties

Outcome Receipts are designed with strong safety guarantees:

1. **No false applied** -- `applied_confirmed` requires a verified receipt linked to an existing event. You cannot accidentally mark something as applied.

2. **Append-only ledger** -- Receipts are never updated or deleted. Every receipt is a permanent record.

3. **Idempotent** -- Sending the same receipt twice is safe. The second submission is ignored.

4. **Monotonic transitions** -- You cannot silently revert from `applied_confirmed` back to `applied_unknown`. Overrides require an explicit audit note and actor ID.

5. **Correlation integrity** -- Receipts referencing an event_id that does not exist are rejected.

---

## Common Scenarios

### Scenario 1: Stripe Checkout Flow

1. Customer completes checkout
2. Stripe sends `checkout.session.completed` webhook
3. HookTunnel captures and forwards it to your app
4. Your app upgrades the user to Pro
5. Your app sends a receipt: `{ "event_id": "...", "status": "processed" }`
6. Dashboard shows: Paid / Delivered / Applied Confirmed

### Scenario 2: Async Processing

1. Your app receives the webhook and immediately returns `200 OK`
2. Your app enqueues the upgrade task
3. Background worker processes the task and upgrades the user
4. Background worker sends an async receipt callback to HookTunnel
5. Dashboard status transitions from Applied Unknown to Applied Confirmed

### Scenario 3: Processing Failure

1. Your app receives the webhook
2. Your app tries to upgrade the user but hits a database error
3. Your app sends a receipt with `status: "failed"`
4. Dashboard shows: Paid / Delivered / Applied Failed
5. You can [replay](/replay/) the webhook after fixing the issue

---

## Getting Started

1. **Create a hook** and configure your provider
2. **Choose a receipt method** -- response header, response body, or async callback
3. **Set up signing** for async callbacks -- see [Receipt Signing](/receipt-signing/)
4. **Monitor the dashboard** -- watch events transition from Unknown to Confirmed
5. **Investigate gaps** -- use [Reconciliation](/reconciliation/) to find unconfirmed payments

---

## What's Next?

- [Receipt Signing](/receipt-signing/) -- Authenticate your receipt callbacks
- [Webhook Replay](/replay/) -- Re-send events that were not applied
- [Stripe Reconciliation](/reconciliation/) -- Find payments without confirmed outcomes
