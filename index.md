---
layout: default
title: HookTunnel Documentation
description: Learn how to use HookTunnel for webhook debugging, testing, and verified business outcomes
---

# HookTunnel Documentation

Welcome to the official HookTunnel documentation. Learn how to capture, inspect, debug, and replay webhooks -- and prove that your app actually applied the business changes your customers paid for.

---

## What Is HookTunnel?

HookTunnel is a webhook inspection and reliability platform. Point your webhook providers (Stripe, Twilio, GitHub, etc.) at a HookTunnel URL and instantly see every event, debug failures, replay missed webhooks, and prove that payments resulted in the right business outcomes.

HookTunnel goes beyond basic webhook capture. It answers the question that keeps founders up at night: **"Stripe says they paid -- but did my app actually grant access?"**

---

## Quick Start: Create Your First Webhook Endpoint

```
1. Sign up at hooktunnel.com
2. Click "Create Hook" in the dashboard
3. Copy your unique webhook URL (e.g., https://hooks.hooktunnel.com/h/abc123...)
4. Paste it into your provider's webhook settings
5. Watch events appear in real-time
```

Your webhook URL looks like this:

```
https://hooks.hooktunnel.com/h/<your-hook-id>
```

Every HTTP request sent to that URL is captured, logged, and available for inspection in your dashboard.

---

## Dashboard Overview

Your HookTunnel dashboard at `app.hooktunnel.com` gives you:

- **Hook List** -- All your webhook endpoints with status, provider, and request counts
- **Event Stream** -- Real-time view of incoming webhook events
- **Event Detail** -- Full headers, body, and metadata for each captured request
- **Smart Search** -- AI-powered semantic search to find events by meaning
- **Investigations** -- Track and resolve webhook issues with pattern matching
- **Replay** -- Re-send captured webhooks to test fixes (Pro+)
- **Outcome Status** -- See Paid / Delivered / Applied status for payment events (Pro+)
- **Reconciliation** -- Compare Stripe payments against confirmed outcomes (Pro+)

---

## Features by Plan

| Feature | Free | Pro | Team | Enterprise |
|---|:---:|:---:|:---:|:---:|
| Webhook endpoints | 1 | 10 | 50 | 1,000 |
| Requests per day | 100 | Unlimited | Unlimited | Unlimited |
| Event history | 24 hours | 30 days | 90 days | 1 year |
| Smart Search | Yes | Yes | Yes | Yes |
| Find Similar | Yes | Yes | Yes | Yes |
| Investigations | Yes | Yes | Yes | Yes |
| Live Monitor | Yes | Yes | Yes | Yes |
| CLI Tool | Yes | Yes | Yes | Yes |
| Test Your Webhook | Yes | Yes | Yes | Yes |
| Request Replay | -- | Yes | Yes | Yes |
| Outcome Receipts | -- | Yes | Yes | Yes |
| Reconciliation | -- | Yes | Yes | Yes |
| Batch Replay | -- | Yes | Yes | Yes |
| Shadow/Mirror Delivery | -- | -- | Yes | Yes |
| Scheduled Reconciliation | -- | -- | -- | Yes |

---

## Documentation

### Core Features

- **[Webhook Inspection](/webhook-inspection/)** -- Capture and inspect webhook events from any provider
- **[Test Your Webhook](/test-webhook/)** -- Send test events to verify your hook works end-to-end
- **[Smart Search](/features/semantic-search/)** -- Find webhooks by meaning, not just text
- **[Smart Investigations](/features/investigations/)** -- Track and resolve webhook issues
- **[CLI Tool](/features/cli/)** -- Forward webhooks to localhost for development

### Pro Features

- **[Webhook Replay](/replay/)** -- Re-send captured webhooks with safety guardrails
- **[Outcome Receipts](/outcome-receipts/)** -- Prove your app applied business changes
- **[Receipt Signing](/receipt-signing/)** -- Authenticate receipt callbacks with HMAC-SHA256
- **[Stripe Reconciliation](/reconciliation/)** -- Compare payments against confirmed outcomes
- **[Shadow/Mirror Delivery](/mirror-delivery/)** -- Test new endpoints without affecting production

### Reference

- **[API Reference](/api-reference/)** -- Programmatic access to all HookTunnel features

---

## Need Help?

- **Bug reports**: [GitHub Issues](https://github.com/Hulupeep/hooktunnel-web/issues)
- **Feature requests**: [GitHub Discussions](https://github.com/Hulupeep/hooktunnel-web/discussions)
- **Security issues**: security@hooktunnel.com
