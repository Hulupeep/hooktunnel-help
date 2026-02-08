---
layout: default
title: Shadow/Mirror Delivery
description: Test new endpoints alongside production without affecting live traffic
permalink: /mirror-delivery/
---

# Shadow/Mirror Delivery (Team+)

**The problem:** You built a new version of your webhook handler and you want to test it with real traffic. But you cannot just switch over -- if the new handler has a bug, you will miss payments, lose data, or break your customers' experience. You need to test with real events without any risk to production.

Shadow delivery solves this. HookTunnel sends every incoming webhook to **two endpoints simultaneously**: your production handler (primary) and your new handler (shadow). The shadow target's response has no effect on the production flow.

**Shadow/Mirror delivery requires a Team plan or higher.**

---

## How It Works

```
Provider sends webhook
        |
        v
HookTunnel receives event
        |
        +---> Primary target (production)  --> Response logged
        |
        +---> Shadow target (staging/new)   --> Response logged separately
```

- The **primary target** is your production webhook handler. Its response determines the delivery status of the event.
- The **shadow target** is your new or staging handler. Its response is logged for comparison but does **not** affect the event's delivery status.

Both targets receive the exact same request (method, headers, body). Both responses are captured so you can compare them side by side.

---

## Setting Up a Shadow Target

### From the Dashboard

1. Open your hook's detail page
2. Go to **Settings**
3. Find the **Shadow/Mirror Target** section
4. Enter the URL of your shadow endpoint (e.g., `https://staging.myapp.com/webhook`)
5. Save

### From the API

Set the `shadow_target` field on your hook:

```bash
curl -X PATCH https://api.hooktunnel.com/api/hooks/<hookId> \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"shadow_target": "https://staging.myapp.com/webhook"}'
```

To disable shadow delivery, set `shadow_target` to `null`:

```bash
curl -X PATCH https://api.hooktunnel.com/api/hooks/<hookId> \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"shadow_target": null}'
```

---

## Viewing Shadow Results

When shadow delivery is active, each event in the dashboard shows two response columns:

| | Primary (Production) | Shadow (Staging) |
|---|---|---|
| **Status** | 200 | 200 |
| **Duration** | 145ms | 320ms |
| **Body** | `{"received": true}` | `{"received": true, "version": "2.0"}` |

This makes it easy to spot differences between your current and new handler:

- **Status code mismatches** -- Your new handler is returning errors for events the old one handles fine
- **Response time differences** -- Your new handler is significantly slower or faster
- **Response body differences** -- Your new handler produces different output

---

## Use Cases

### Validating a New Handler Version

1. Deploy your new webhook handler to a staging URL
2. Set it as the shadow target on your production hook
3. Let real traffic flow to both handlers for a few hours or days
4. Compare responses in the dashboard
5. When you are confident the new handler works correctly, update your primary target

### A/B Testing Webhook Processing

1. Set up a shadow target pointing to an experimental handler
2. Compare processing results between the two handlers
3. Decide which approach to keep based on real-world data

### Load Testing with Real Traffic

1. Point the shadow target at your new infrastructure
2. Monitor its performance under real traffic patterns
3. Identify bottlenecks before switching production traffic

### Migrating Between Cloud Providers

1. Set up your webhook handler on the new cloud provider
2. Use shadow delivery to validate it receives and processes events correctly
3. Once validated, switch your primary target to the new provider

---

## Important Notes

### Shadow Responses Do Not Affect Production

The shadow target's response (status code, body, errors) has absolutely no effect on:
- The event's delivery status
- Whether the provider considers the webhook delivered
- Receipt processing
- Replay logic

The shadow response is logged purely for your comparison and analysis.

### Both Targets Receive the Same Request

The shadow target receives an exact copy of the request sent to the primary target, including:
- HTTP method
- All headers (including signature headers)
- Complete request body

### Shadow Delivery Adds Latency

Shadow delivery happens asynchronously -- it does not slow down the primary delivery. However, it does double the number of outbound requests from HookTunnel for each incoming event.

### Shadow Target Errors Are Not Alerts

If your shadow target returns errors or is unreachable, this is logged but does not trigger any alerts. The shadow target is treated as "best effort" -- failures are expected during testing.

---

## Common Issues

### "Shadow target not receiving events"

1. Verify the shadow target URL is correct and accessible from the internet
2. Check that the URL does not have authentication that blocks HookTunnel
3. Make sure your Team plan is active

### "Shadow target receiving events but returning 401/403"

Your shadow target likely has authentication or IP restrictions. Make sure it accepts requests from HookTunnel's infrastructure. Alternatively, set up a shared secret or API key that both targets accept.

---

## What's Next?

- [Webhook Inspection](/webhook-inspection/) -- Understand what your provider is sending
- [Webhook Replay](/replay/) -- Re-send specific events to either target
- [Outcome Receipts](/outcome-receipts/) -- Track business outcomes for both handlers
