---
layout: default
title: Test Your Webhook
description: Send test events to verify your webhook endpoint works end-to-end
permalink: /test-webhook/
---

# Test Your Webhook

**The problem:** You have created a webhook endpoint but you are not sure it is working. You do not want to wait for a real event from your provider to find out.

HookTunnel lets you send realistic test events through the real ingress pipeline so you can verify your hook works end-to-end before going live.

---

## How It Works

Test events are **not** synthetic database inserts. They flow through the exact same code path as real webhooks:

```
Dashboard "Send Test Event" button
        |
        v
HookTunnel API sends POST to hooks.hooktunnel.com/h/<hookId>
        |
        v
Data plane ingress receives, logs, and processes it
        |
        v
Event appears in your dashboard (marked as test)
```

This means if a test event appears in your event list, you know the entire pipeline is working.

---

## Sending a Test Event

### Step by Step

1. Open the dashboard at [app.hooktunnel.com](https://app.hooktunnel.com)
2. Click on the hook you want to test
3. Click the **Send Test Event** button
4. Select a provider from the dropdown:
   - **Generic** -- A standard webhook payload
   - **Stripe** -- Simulated Stripe event
   - **Twilio** -- Simulated Twilio callback
5. Select an event type (e.g., "Checkout Completed" for Stripe)
6. Click **Send**
7. The event appears in your event list within seconds

### Available Test Payloads

| Provider | Event Type | Description |
|----------|-----------|-------------|
| Generic | Test Event | A sample order payload with amount, currency, and items |
| Stripe | Checkout Completed | Simulates a `checkout.session.completed` event |
| Stripe | Payment Succeeded | Simulates an `invoice.payment_succeeded` event |
| Twilio | Call Completed | Simulates a voice call status callback |
| Twilio | SMS Received | Simulates an inbound SMS webhook |

---

## Identifying Test Events

Test events include special markers so you can always distinguish them from real production webhooks:

### Header Marker

Every test event includes the header:

```
X-HookTunnel-Test: true
```

Along with metadata headers:

```
X-HookTunnel-Test-Provider: stripe
X-HookTunnel-Test-Event: checkout.session.completed
```

### Payload Marker

Every test payload includes a `_hooktunnel` object:

```json
{
  "type": "checkout.session.completed",
  "data": { ... },
  "_hooktunnel": {
    "test": true,
    "provider": "stripe",
    "event_type": "checkout.session.completed",
    "sent_at": "2026-02-08T12:00:00.000Z"
  }
}
```

You can use these markers in your own application to skip processing test events in production, or to run them through a separate test path.

---

## Rate Limits

Test events are rate-limited to **5 per minute** per user. This prevents accidental abuse while giving you plenty of room for testing.

If you hit the limit, wait a moment and try again. The response headers tell you when the limit resets:

```
X-RateLimit-Remaining: 0
Retry-After: 45
```

---

## Common Scenarios

### Verifying a New Hook

After creating a new hook, send a test event to confirm everything is wired up:

1. Create the hook
2. Send a Generic test event
3. Check the event list -- if it appears, you are ready to configure your provider

### Testing Provider-Specific Handling

If your application processes Stripe and Twilio webhooks differently, send test events for each:

1. Send a Stripe "Checkout Completed" test event
2. Send a Twilio "SMS Received" test event
3. Verify both appear correctly in your dashboard with the right provider metadata

### Testing with the CLI

If you are using the [CLI tool](/features/cli/) to forward webhooks to localhost:

1. Start the CLI: `hooktunnel connect dev 3000`
2. Send a test event from the dashboard
3. Watch the event arrive in your terminal and hit your local server
4. Verify your local handler processes it correctly

---

## Common Issues

### "Test event timed out"

This means the data plane ingress did not respond within 10 seconds. Possible causes:
- The data plane service is temporarily unavailable
- Network connectivity issues between the control plane and data plane

Wait a moment and try again. If the problem persists, check the [HookTunnel status page](https://hooktunnel.com).

### "Authentication required"

You must be logged in to send test events. This prevents unauthorized users from flooding endpoints.

### "Hook not found"

The hook ID does not exist or has been deleted. Verify you are on the correct hook detail page.
