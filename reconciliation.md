---
layout: default
title: Stripe Reconciliation
description: Compare Stripe payments against confirmed business outcomes to find revenue gaps
permalink: /reconciliation/
---

# Stripe Reconciliation

**The problem:** Stripe says 47 customers paid this week. Your app says 45 users have Pro access. Where are the missing 2? Which payments did not result in the right business outcome? And how do you fix it without causing more problems?

HookTunnel's reconciliation dashboard answers this in seconds: it shows every Stripe payment event alongside its outcome status, highlights the gaps, and gives you safe actions to close them.

**Reconciliation requires a Pro plan or higher.**

---

## How Reconciliation Works

Reconciliation compares two independent data sources:

| Source | Data | Question |
|--------|------|----------|
| **Stripe Events** (captured by HookTunnel) | Payment intents, checkout sessions, invoices | "What was paid?" |
| **Outcome Receipts** (from your app) | Applied confirmations | "What was applied?" |

The reconciliation view shows you where these two sources agree (everything is fine) and where they disagree (you have a gap that needs attention).

---

## Reconciliation Buckets

When you open the reconciliation dashboard and select a time range, events are sorted into buckets:

| Bucket | Meaning | Action Needed? |
|--------|---------|----------------|
| **Applied Confirmed** | Paid and your app confirmed it applied the change | No -- this is the happy path |
| **Applied Unknown** | Paid and delivered, but your app has not confirmed | **Yes -- this is the revenue risk zone** |
| **Applied Failed** | Paid and delivered, but your app reported failure | Yes -- investigate why |
| **Not Delivered** | Paid but the webhook was never delivered to your app | Yes -- replay needed |

The **Applied Unknown** bucket is highlighted as a revenue risk. These are payments where Stripe collected money but you cannot prove the customer received what they paid for.

---

## Using the Reconciliation Dashboard

### Step 1: Select a Time Range

1. Navigate to the reconciliation page in your dashboard
2. Use the date picker to select a time range (e.g., "Last 7 days")
3. Bucket counts update to show totals for the selected period

### Step 2: Review the Buckets

The dashboard shows a summary bar with counts for each bucket. A typical view might look like:

```
Last 7 Days

Applied Confirmed: 45    Applied Unknown: 2    Applied Failed: 0    Not Delivered: 0
      (green)              (amber/warning)         (red)                (red)
```

### Step 3: Drill Down into Gaps

Click on any bucket to see the individual events. Each gap row shows:

- **Stripe Event ID** -- The Stripe object (payment intent, session, invoice)
- **Customer** -- Customer email or ID (if available in the event data)
- **Amount** -- Payment amount and currency
- **Delivery Status** -- Whether the webhook was delivered to your app
- **Processing Status** -- Current outcome state
- **Timestamp** -- When the payment occurred

### Step 4: Take Action on Gaps

Each gap row has action buttons:

| Action | What It Does |
|--------|-------------|
| **Replay** | Re-sends the webhook to your app via the [replay system](/replay/) |
| **Open Trace** | Shows the full event trace: delivery attempts, responses, receipts |
| **Mark Resolved** | Closes the gap with an audit note (e.g., "manually verified in Stripe dashboard") |

---

## Gap Actions in Detail

### Replay

Clicking **Replay** on a gap event creates a replay job with the standard [replay guardrails](/replay/). The event is re-sent to your app, and if your app sends a receipt, the gap closes automatically.

### Open Trace

The trace view shows the complete timeline for the event:

```
1. Stripe payment succeeded (checkout.session.completed)
2. HookTunnel received event at 12:00:01
3. Forwarded to https://myapp.com/webhook at 12:00:02
4. Your app responded 200 OK at 12:00:02 (150ms)
5. No receipt received (SLA: 5 minutes)
6. Status: APPLIED_UNKNOWN
```

This helps you diagnose why your app did not send a receipt. Common causes:
- Your app processes webhooks asynchronously and the receipt callback is not implemented
- Your app processed the event but the receipt failed to send (network issue)
- Your app failed silently without reporting an error

### Mark Resolved

If you manually verify the outcome outside of HookTunnel (for example, you check the Stripe dashboard and the customer's account directly), you can mark the gap as resolved.

**Marking resolved requires an audit note.** You must explain why you are closing this gap. Examples:

- "Verified manually in Stripe dashboard -- customer has Pro access"
- "Customer was refunded, no action needed"
- "Duplicate event from Stripe -- original was processed correctly"

The audit note, along with your user ID and timestamp, is permanently recorded.

---

## Resolution Workflow

A typical reconciliation workflow:

1. **Daily check** -- Open reconciliation, select "Last 24 hours"
2. **Review gaps** -- Any new Applied Unknown or Not Delivered events?
3. **For Applied Unknown** -- Check if your app processed it but did not send a receipt. If so, send a manual receipt or fix your receipt pipeline.
4. **For Not Delivered** -- Click Replay to re-send the webhook
5. **For already handled** -- Click Mark Resolved with an audit note
6. **Verify** -- Check that the Applied Confirmed count matches your Stripe payment count

---

## Data Sources

### Stripe as Source of Truth for "Paid"

The reconciliation paid count comes from the `stripe_events` table -- the Stripe webhook events that HookTunnel has captured. HookTunnel does not call the Stripe API during reconciliation; it only uses the events it has already received.

This means if HookTunnel did not receive a Stripe event (for example, the webhook was misconfigured), that payment will not appear in reconciliation. Monitor your Stripe webhook delivery rate separately to ensure coverage.

### Receipts as Source of Truth for "Applied"

The applied count comes from the receipts table. An event is counted as "applied confirmed" only if a verified receipt exists for it. HookTunnel does not infer application status from HTTP response codes or response body content (unless you explicitly include a receipt in the response).

---

## Common Scenarios

### Weekly Finance Reconciliation

1. At the end of each week, open the reconciliation dashboard
2. Select the past 7 days
3. Verify that Applied Confirmed matches Stripe's payment count
4. Investigate and resolve any gaps
5. Export the results for your records (Enterprise feature)

### Post-Outage Recovery

1. Your server was down during a payment spike
2. Open reconciliation for the outage window
3. See how many payments landed in "Not Delivered" or "Applied Unknown"
4. Use batch replay to re-send all missed events
5. Monitor receipts to confirm they were applied
6. Close any remaining gaps with Mark Resolved

### New Receipt Integration Verification

1. You just implemented receipt callbacks in your app
2. Open reconciliation for the past 24 hours
3. Verify that new payments are moving from "Applied Unknown" to "Applied Confirmed"
4. If the count is not increasing, check your receipt signing and callback URL

---

## Common Issues

### "All events show as Applied Unknown"

You have not implemented receipt callbacks yet. Your app receives webhooks but does not tell HookTunnel what it did with them. See [Outcome Receipts](/outcome-receipts/) to get started.

### "Paid count does not match Stripe dashboard"

HookTunnel can only count Stripe events it has received. If your Stripe webhook endpoint was misconfigured or HookTunnel was unreachable, some events may be missing. Check Stripe's webhook delivery logs for failed attempts.

### "Cannot mark as resolved"

Resolving a gap requires an audit note. You must type a reason before the button becomes active.

---

## What's Next?

- [Outcome Receipts](/outcome-receipts/) -- Set up receipts so your app can confirm outcomes
- [Webhook Replay](/replay/) -- Re-send events to close gaps
- [Receipt Signing](/receipt-signing/) -- Secure your receipt callbacks
