---
layout: default
title: Webhook Replay
description: Re-send captured webhooks with safety guardrails to recover from failures
permalink: /replay/
---

# Webhook Replay

**The problem:** A webhook failed -- maybe your server was down, maybe your handler had a bug. The provider already sent the event. You need to re-process it, but you do not want to accidentally double-charge a customer or send a duplicate email.

HookTunnel's replay system lets you re-send any captured webhook with built-in safety guardrails that protect against common replay dangers.

**Replay requires a Pro plan or higher.**

---

## Single Event Replay

### How to Replay from the Dashboard

1. Open any hook's detail page
2. Find the event you want to replay in the event list
3. Click the **Replay** button
4. A confirmation dialog appears showing:
   - The original event details (method, path, headers)
   - Replay target (where it will be sent)
   - Signature mode (see below)
5. Confirm to execute the replay

### How to Replay from the API

```bash
curl -X POST https://api.hooktunnel.com/api/replay \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "logId": "log_abc123",
    "targetUrl": "http://localhost:3000/webhook",
    "targetType": "dev",
    "signatureMode": "strip"
  }'
```

Response:

```json
{
  "success": true,
  "replay": {
    "jobId": "job_xyz789",
    "originalLogId": "log_abc123",
    "targetUrl": "http://localhost:3000/webhook",
    "targetType": "dev",
    "signatureMode": "strip",
    "method": "POST",
    "responseStatus": 200,
    "duration": 145,
    "status": "success",
    "replayedAt": "2026-02-08T12:10:00Z"
  }
}
```

### How to Replay from the CLI

```bash
# Replay to your connected tunnel
hooktunnel replay log_abc123

# Replay to a specific URL
hooktunnel replay log_abc123 --to http://localhost:4000/webhook
```

---

## Signature Modes

When replaying a webhook, the original provider's signature headers (like `stripe-signature` or `x-twilio-signature`) may no longer be valid. HookTunnel gives you three options:

| Mode | What It Does | When to Use |
|------|-------------|-------------|
| `strip` | Removes all signature headers | Default. Use when your handler can skip signature verification in dev/test |
| `preserve_original` | Keeps the original signature headers unchanged | Use when you need to test signature verification logic (note: signatures will likely fail verification since the payload was replayed, not sent fresh by the provider) |
| `resign` | Re-signs with the hook's current secret | Use when your handler requires valid signatures |

### Replay Headers

Every replayed request includes these additional headers:

```
X-HookTunnel-Replay: true
X-HookTunnel-Original-Id: <original-log-id>
X-HookTunnel-Replay-Job-Id: <replay-job-id>
```

Your application can check for the `X-HookTunnel-Replay: true` header to know a request is a replay rather than a fresh event from the provider.

---

## Receipt-Aware Replay (R8)

If you are using [Outcome Receipts](/outcome-receipts/), replay becomes smarter. Events that have already been confirmed as applied are **excluded from replay by default**.

### Why This Matters

If an event has `processing_status: applied_confirmed`, replaying it could double-apply the business change (double-provisioning, double-charging, etc.). HookTunnel prevents this automatically.

### Default Behavior

- Events with `applied_confirmed` status: **blocked** from replay
- Events with `applied_unknown` status: **allowed**
- Events with `applied_failed` status: **allowed**
- Events with `not_required` status: **allowed**

### Overriding the Filter

If you have a legitimate reason to replay a confirmed event (for example, you need to re-run it after a data migration), you can override the filter. This requires an **audit note** explaining why:

```bash
curl -X POST https://api.hooktunnel.com/api/replay \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "logId": "log_abc123",
    "include_confirmed": true,
    "audit_note": "Re-running after database migration to backfill new fields"
  }'
```

The audit note is recorded permanently so there is always a record of why a confirmed event was replayed.

---

## Stop on Receipt (R9)

When a replay is in progress and a receipt arrives confirming the event was applied, the replay automatically cancels. This prevents duplicate processing:

```
Replay attempt #1 sent
        |
        v
Your app processes the event and sends a receipt
        |
        v
HookTunnel receives the receipt -> marks event as applied_confirmed
        |
        v
Replay attempt #2 is about to send
        |
        v
HookTunnel checks processing_status -> now applied_confirmed
        |
        v
Replay auto-cancels (reason: receipt_confirmed)
```

Before each replay attempt, the replay engine re-checks the event's processing status. If the event has been confirmed since the replay started, the remaining attempts are cancelled.

---

## Backoff and Rate Limiting (R10)

Replay uses exponential backoff to protect your application from being overwhelmed:

- **First retry**: immediate
- **Second retry**: 1 second delay
- **Third retry**: 2 seconds
- **Fourth retry**: 4 seconds
- And so on, with random jitter to prevent thundering herd effects

Per-destination rate limits also apply. If you are replaying to the same endpoint, HookTunnel spaces out the requests to avoid overloading your server.

---

## Batch Replay with Dry-Run Preview (R11)

For recovering from outages or large-scale failures, you can replay multiple events at once.

### How Batch Replay Works

1. **Define filter criteria** -- Select events by status, time range, or other attributes
2. **Preview (dry run)** -- HookTunnel shows you exactly what will be replayed:
   - Number of events
   - Which processing states they are in
   - Destination endpoint(s)
   - Risk assessment (low/medium/high)
3. **Confirm** -- Click "Run" to execute the batch
4. **Monitor** -- Track progress in the dashboard

### Risk Assessment

The dry-run preview includes a risk assessment:

| Risk Level | Meaning |
|-----------|---------|
| **Low** | All events are `applied_unknown` or `not_delivered`. Safe to replay. |
| **Medium** | Some events have `applied_failed` status. Replay may re-trigger failures. |
| **High** | Includes `applied_confirmed` events (requires explicit override). |

### API

```bash
# Dry-run preview (does not execute)
curl -X POST https://api.hooktunnel.com/api/replay/batch \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "hookId": "abc123",
    "filter": {
      "processing_status": ["applied_unknown", "applied_failed"],
      "time_range": {
        "from": "2026-02-01T00:00:00Z",
        "to": "2026-02-08T00:00:00Z"
      }
    },
    "dry_run": true
  }'
```

---

## Production Replay Safety

Replaying to production requires extra safeguards to prevent accidental damage:

### Requirements

Both conditions must be met:
1. The hook must have `allow_prod_replay: true` set in its configuration
2. The request must include the header `X-Replay-Scope: replay:prod`

If either condition is missing, the replay is blocked and an audit record is created.

### Setting Up Production Replay

```bash
# Enable production replay for a hook (from hook settings)
# Then include the scope header in your replay request:
curl -X POST https://api.hooktunnel.com/api/replay \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-Replay-Scope: replay:prod" \
  -H "Content-Type: application/json" \
  -d '{
    "logId": "log_abc123",
    "targetType": "prod"
  }'
```

---

## Replay Job Lifecycle

Each replay creates a **replay job** with tracked status:

| Status | Meaning |
|--------|---------|
| `pending` | Job created, waiting to execute |
| `running` | Replay request is in flight |
| `completed` | Replay succeeded (target returned 2xx) |
| `failed` | Replay failed (target returned error or timed out) |
| `blocked` | Replay was blocked by a safety gate |
| `cancelled` | Replay was manually or automatically cancelled |

### Viewing Replay History

```bash
# List replay jobs for a hook
curl "https://api.hooktunnel.com/api/replay?hookId=abc123&limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Cancelling a Replay

```bash
curl -X DELETE "https://api.hooktunnel.com/api/replay?jobId=job_xyz789" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Only `pending` and `blocked` jobs can be cancelled.

---

## Common Scenarios

### Recovering from a Server Outage

1. Your server was down for 30 minutes
2. During that time, 15 webhooks were received but delivery failed
3. Open your hook in the dashboard
4. Filter events by the outage time window
5. Use batch replay to re-send all failed events
6. Monitor receipts to confirm they were all applied

### Debugging a Handler Bug

1. A webhook arrived and your handler returned 500
2. You fix the bug in your code
3. Start the CLI: `hooktunnel connect dev 3000`
4. Replay the failed event to your local server
5. Verify the fix works before deploying

### Re-Running After a Database Migration

1. You migrated your database schema
2. Some older events need to be re-processed to populate new fields
3. Use batch replay with the appropriate time range
4. Include `include_confirmed: true` with an audit note explaining the migration

---

## Common Issues

### "Replay is a Pro feature"

Replay requires a Pro plan or higher. Upgrade at [hooktunnel.com/pricing](https://hooktunnel.com/#pricing).

### "Event already confirmed"

The event has `applied_confirmed` status. To replay it anyway, set `include_confirmed: true` and provide an `audit_note`.

### "Production replay is blocked"

You need both: the hook's `allow_prod_replay` setting enabled, and the `X-Replay-Scope: replay:prod` header in your request.

### "Request log has no stored body"

The original request body was not stored (possibly because the event was received on a Free plan without payload storage). You cannot replay without the original body.

---

## What's Next?

- [Outcome Receipts](/outcome-receipts/) -- Enable receipts to make replay receipt-aware
- [Stripe Reconciliation](/reconciliation/) -- Find all events that need replay
- [API Reference](/api-reference/) -- Full replay API details
