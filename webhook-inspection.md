---
layout: default
title: Webhook Inspection
description: Capture and inspect webhook events from Stripe, Twilio, GitHub, and any provider
permalink: /webhook-inspection/
---

# Webhook Inspection

**The problem:** You are integrating with a webhook provider (Stripe, Twilio, GitHub, etc.) and you cannot see what they are actually sending. You are debugging blind.

HookTunnel gives you a unique URL that captures every webhook request, so you can inspect headers, bodies, and metadata in real time.

---

## How It Works

1. **Create a hook** -- HookTunnel generates a unique URL for you
2. **Point your provider** -- Configure Stripe, Twilio, or any provider to send webhooks to your HookTunnel URL
3. **Inspect events** -- Every request appears in your dashboard with full details

```
Provider sends webhook
        |
        v
https://hooks.hooktunnel.com/h/<your-hook-id>
        |
        v
HookTunnel captures the request (headers, body, metadata)
        |
        v
You inspect it in the dashboard at app.hooktunnel.com
```

---

## Creating a Webhook Endpoint

### From the Dashboard

1. Log in at [app.hooktunnel.com](https://app.hooktunnel.com)
2. Click **Create Hook**
3. Select a provider type: Generic, Stripe, or Twilio
4. Copy the generated webhook URL

### From the CLI

```bash
hooktunnel hooks  # List your current hooks
```

### From the API

```bash
curl -X POST https://api.hooktunnel.com/api/hooks \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"provider": "stripe"}'
```

Response:

```json
{
  "hook_id": "abc123def456ghi789",
  "hook_url": "https://hooks.hooktunnel.com/h/abc123def456ghi789",
  "provider": "stripe",
  "created_at": "2026-02-08T12:00:00Z"
}
```

---

## Viewing Captured Requests

Every captured webhook shows you:

### Headers

The full HTTP headers sent by the provider, including:
- `Content-Type`
- Authentication/signature headers (e.g., `stripe-signature`, `x-twilio-signature`)
- Custom headers from the provider

### Body

The complete request body, formatted for readability:
- JSON payloads are syntax-highlighted and collapsible
- Form-encoded data is parsed and displayed as key-value pairs
- Raw payloads are shown as-is

### Metadata

Additional context captured by HookTunnel:
- **Timestamp** -- When the request was received
- **Method** -- HTTP method (POST, GET, etc.)
- **Source IP** -- Where the request came from
- **Content length** -- Size of the payload
- **Response status** -- What HookTunnel returned to the provider

---

## Filtering and Searching Events

### Basic Filtering

Use the filter controls in the event list to narrow down results by:
- HTTP method
- Status code
- Time range
- Provider type

### Smart Search

HookTunnel includes AI-powered semantic search. Instead of searching for exact text, you can search by meaning:

| You type | What it finds |
|----------|---------------|
| `stripe payment failed` | Stripe payment failure webhooks |
| `POST checkout` | POST requests to checkout-related paths |
| `twilio sms` | Twilio SMS-related webhook events |

See the full [Smart Search guide](/features/semantic-search/) for details.

### Find Similar

Click any event and then click **Similar** to find requests that share the same patterns. This is useful for:
- Finding all instances of a recurring error
- Grouping related events from a checkout flow
- Spotting anomalies by seeing what "normal" looks like

---

## Provider Setup Guides

### Stripe

1. Go to [Stripe Dashboard > Developers > Webhooks](https://dashboard.stripe.com/webhooks)
2. Click **Add endpoint**
3. Paste your HookTunnel URL: `https://hooks.hooktunnel.com/h/<your-hook-id>`
4. Select the events you want to receive (e.g., `checkout.session.completed`, `invoice.payment_succeeded`)
5. Click **Add endpoint**

Stripe will start sending events to your HookTunnel URL immediately. You can verify it is working by clicking **Send test webhook** in the Stripe dashboard.

### Twilio

1. Go to your Twilio console
2. Navigate to the resource you want to monitor (Phone Number, Messaging Service, etc.)
3. In the webhook/callback URL field, paste: `https://hooks.hooktunnel.com/h/<your-hook-id>`
4. Save your changes

For voice calls, set the **Status Callback URL** to your HookTunnel URL to receive call status events.

### GitHub

1. Go to your GitHub repository > **Settings** > **Webhooks**
2. Click **Add webhook**
3. Set the Payload URL to: `https://hooks.hooktunnel.com/h/<your-hook-id>`
4. Set Content type to `application/json`
5. Select the events you want to receive
6. Click **Add webhook**

### Generic (Any Provider)

Any service that can send HTTP requests can use HookTunnel. Just point the webhook/callback URL to:

```
https://hooks.hooktunnel.com/h/<your-hook-id>
```

HookTunnel accepts any HTTP method (POST, PUT, PATCH, DELETE, GET) and any content type.

---

## Live Event Monitor

Watch events stream in real-time:

1. Go to any hook's detail page
2. Click the **Live** view mode button
3. Events appear as they arrive (3-second polling)
4. Click any event to view details or start an investigation

Features:
- **Pause/Resume** -- Stop auto-refresh when analyzing an event
- **New event highlighting** -- Fresh events flash blue
- **Auto-scroll** -- Stays at the top when live

---

## Test Your Webhook

Before configuring your real provider, you can send test events to verify your hook works.

1. Open any hook in the dashboard
2. Click **Send Test Event**
3. Select a provider and event type (e.g., Stripe > Checkout Completed)
4. Click **Send**

Test events flow through the real ingress pipeline -- the same path as production webhooks. They are marked with an `X-HookTunnel-Test` header and a `_hooktunnel.test` flag in the payload so you can distinguish them from real events.

Available test payloads:
- **Generic** -- Test Event (order payload)
- **Stripe** -- Checkout Completed, Payment Succeeded
- **Twilio** -- Call Completed, SMS Received

See the full [Test Your Webhook guide](/test-webhook/) for details.

---

## Event History and Retention

How long your events are stored depends on your plan:

| Plan | History |
|------|---------|
| Free | 24 hours |
| Pro | 30 days |
| Team | 90 days |
| Enterprise | 1 year |

Events older than your plan's retention window are automatically removed. To keep events longer, upgrade your plan.

---

## Common Issues

### "No events appearing"

1. Verify your provider is configured with the correct HookTunnel URL
2. Check that your provider is actually sending events (trigger a test action)
3. Make sure your hook is in `active` status (not paused or deleted)
4. If you are on the Free plan, check that you have not exceeded the 100 requests/day limit

### "Events are appearing but I cannot see the body"

Full payload storage requires a Pro plan or higher. Free tier events capture headers and metadata but may not store the complete body.

### "Provider says webhook delivery failed"

HookTunnel always returns a `200 OK` to the provider to prevent retries. If your provider reports a delivery failure, check:
1. Your HookTunnel URL is correctly formatted
2. There are no typos in the hook ID
3. The hook has not been deleted

---

## What's Next?

- [Replay a captured webhook](/replay/) to test your handler
- [Set up outcome receipts](/outcome-receipts/) to prove your app applied business changes
- [Use the CLI](/features/cli/) to forward webhooks to localhost
