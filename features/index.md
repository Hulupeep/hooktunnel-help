---
layout: default
title: Features
description: Explore HookTunnel's powerful webhook debugging and reliability features
permalink: /features/
---

# Features

HookTunnel is packed with features designed to make webhook development, debugging, and reliability effortless.

---

## Core Features (All Plans)

### [Webhook Inspection](/webhook-inspection/)
Capture and inspect every webhook event from any provider. See full headers, bodies, and metadata in real time.

- Unique URLs for Stripe, Twilio, GitHub, and any provider
- Real-time event streaming
- Full header and body inspection

### [Test Your Webhook](/test-webhook/)
Send realistic test events through the real ingress pipeline to verify your hook works end-to-end.

- Provider-specific payloads (Stripe, Twilio, generic)
- Flows through the same path as real webhooks
- Test events are clearly marked in the dashboard

### [Smart Search](/features/semantic-search/)
Find webhooks by meaning, not just text. Use AI-powered semantic search to locate requests by intent and discover similar patterns.

- Search by concepts like "stripe payment failed"
- Find similar requests with one click
- Works offline in your browser

### [Smart Investigations](/features/investigations/)
Track and resolve webhook issues with AI-powered pattern matching. Build institutional knowledge about recurring problems.

- Auto-link related events automatically
- AI-suggested causes from pattern memory
- Anomaly scoring for severity triage
- Live event monitoring

### [CLI Tool](/features/cli/)
Forward webhooks to your local development server with the HookTunnel CLI.

- `npx hooktunnel-cli connect dev 3000`
- Real-time request logging in terminal
- Works with any local server

---

## Pro Features

### [Webhook Replay](/replay/)
Re-send any captured webhook with built-in safety guardrails.

- Single event and batch replay
- Receipt-aware filtering (confirmed events excluded by default)
- Stop-on-receipt (auto-cancel when receipt arrives)
- Exponential backoff and rate limiting
- Production replay safety gates

### [Outcome Receipts](/outcome-receipts/)
Prove your app applied business changes, not just that a webhook was delivered.

- Three receipt methods: response header, response body, async callback
- Paid / Delivered / Applied status tracking
- Append-only receipt ledger with audit trail
- HMAC-SHA256 authentication

### [Receipt Signing](/receipt-signing/)
Authenticate receipt callbacks with per-hook HMAC-SHA256 signing secrets.

- Per-destination signing secrets
- Key rotation with 24-hour grace period
- Code examples for Node.js, Python, and curl

### [Stripe Reconciliation](/reconciliation/)
Compare Stripe payments against confirmed business outcomes to find revenue gaps.

- Reconciliation buckets (Confirmed, Unknown, Failed, Not Delivered)
- Drill-down into individual gaps
- Gap actions: Replay, Open Trace, Mark Resolved
- Audit trail for all resolutions

---

## Team Features

### [Shadow/Mirror Delivery](/mirror-delivery/)
Test new endpoints alongside production without affecting live traffic.

- Dual delivery to primary and shadow targets
- Side-by-side response comparison
- Zero risk to production traffic

---

## Reference

### [API Reference](/api-reference/)
Programmatic access to all HookTunnel features.

- Hooks, events, replay, receipts, investigations
- Authentication with API keys
- Full request/response examples

---

## Quick Links

- [Getting Started](/) - Set up your first webhook
- [Pricing](https://hooktunnel.com/#pricing) - Compare plans

---

## Feature Requests

Have an idea for a new feature? [Let us know](https://github.com/Hulupeep/hooktunnel-web/issues).
