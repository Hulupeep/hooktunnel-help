---
layout: default
title: API Reference
description: Programmatic access to all HookTunnel features
permalink: /api-reference/
---

# API Reference

HookTunnel provides a REST API for programmatic access to hooks, events, replay, receipts, and reconciliation. All API endpoints are available at `https://api.hooktunnel.com`.

---

## Authentication

All API requests require authentication via an API key. Include your key in the `Authorization` header:

```
Authorization: Bearer YOUR_API_KEY
```

### Getting an API Key

1. Go to [app.hooktunnel.com/settings](https://app.hooktunnel.com/settings)
2. Click **Generate API Key**
3. Copy the key (starts with `ht_`)
4. Store it securely -- it will not be shown again

### API Key Scopes

API keys have the same permissions as your user account. They inherit your plan's tier limits.

---

## Base URL

```
https://api.hooktunnel.com
```

All endpoints below are relative to this base URL. Webhook ingress uses a different domain:

```
https://hooks.hooktunnel.com
```

---

## Common Response Headers

Every API response includes:

| Header | Description |
|--------|-------------|
| `x-request-id` | Unique correlation ID for the request (useful for debugging) |
| `X-RateLimit-Remaining` | Number of requests remaining in the current window |
| `X-RateLimit-Reset` | Unix timestamp when the rate limit resets |

---

## Hooks

### Create Hook

```
POST /api/hooks
```

Creates a new webhook endpoint.

**Request body:**

```json
{
  "provider": "stripe"
}
```

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `provider` | string | No | `generic` (default), `stripe`, `twilio` |

**Response (200):**

```json
{
  "hook_id": "abc123def456ghi789",
  "hook_url": "https://hooks.hooktunnel.com/h/abc123def456ghi789",
  "provider": "stripe",
  "created_at": "2026-02-08T12:00:00Z"
}
```

**Error (403) -- Plan limit reached:**

```json
{
  "error": {
    "code": "PLAN_LIMIT",
    "message": "You've reached your plan's hook limit (1). Upgrade to create more hooks.",
    "tier": "free",
    "limit_key": "max_hooks",
    "limit": 1,
    "current": 1,
    "upgrade_url": "/pricing?reason=hook_limit"
  }
}
```

### List Hooks

```
GET /api/hooks
```

Returns all hooks for the authenticated user.

**Response (200):**

```json
{
  "hooks": [
    {
      "id": "uuid",
      "hook_id": "abc123def456ghi789",
      "hook_url": "https://hooks.hooktunnel.com/h/abc123def456ghi789",
      "provider": "stripe",
      "env": "production",
      "status": "active",
      "created_at": "2026-02-08T12:00:00Z",
      "request_count": 142
    }
  ],
  "total": 1,
  "tier": "pro"
}
```

---

## Event Logs

### List Events for a Hook

```
GET /api/hooks/:hookId/logs
```

Returns captured webhook events for a specific hook.

**Query parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | number | Max events to return (default: 50, max: 200) |
| `offset` | number | Pagination offset |

**Response (200):**

```json
{
  "logs": [
    {
      "id": "log_abc123",
      "hook_id": "abc123def456",
      "method": "POST",
      "path": "/h/abc123def456",
      "headers": { "content-type": "application/json", "stripe-signature": "..." },
      "body": "{...}",
      "source_ip": "54.187.174.169",
      "received_at": "2026-02-08T12:00:01Z",
      "response_status": 200,
      "processing_status": "applied_confirmed"
    }
  ],
  "meta": {
    "tier": "pro",
    "history_days": 30,
    "history_cutoff": "2026-01-09T12:00:00Z",
    "filtered": true
  }
}
```

History retention is enforced by tier. Free tier logs older than 24 hours are not returned.

---

## Send Test Event

### Send Test Webhook

```
POST /api/hooks/:hookId/test
```

Sends a test event through the real ingress pipeline.

**Request body:**

```json
{
  "provider": "stripe",
  "eventType": "checkout.session.completed"
}
```

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `provider` | string | No | `generic` (default), `stripe`, `twilio` |
| `eventType` | string | No | Depends on provider (see below) |

**Event types by provider:**

| Provider | Event Types |
|----------|------------|
| `generic` | `webhook.test` |
| `stripe` | `checkout.session.completed`, `invoice.payment_succeeded` |
| `twilio` | `voice.call_completed`, `sms.received` |

**Response (200):**

```json
{
  "success": true,
  "provider": "stripe",
  "eventType": "checkout.session.completed",
  "responseStatus": 200,
  "requestId": "corr-abc123"
}
```

**Rate limit:** 5 test events per minute per user.

---

## Receipts

### Submit Receipt

```
POST /api/v1/receipts
```

Submit an outcome receipt for a captured event. Requires HMAC-SHA256 signature.

**Headers:**

```
Content-Type: application/json
X-HookTunnel-Signature: sha256=<computed-hmac>
```

**Request body:**

```json
{
  "event_id": "evt_abc123",
  "status": "processed",
  "schema_version": "1.0",
  "ref": "user_123_pro_upgrade",
  "timestamp": "2026-02-08T12:05:00Z"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `event_id` | string | Yes | The ID of the captured webhook event |
| `status` | string | Yes | `processed`, `failed`, or `queued` |
| `schema_version` | string | Yes | Always `"1.0"` |
| `ref` | string | No | Your internal reference (e.g., user ID, order ID) |
| `timestamp` | string | No | ISO 8601 timestamp of when the change was applied |

**Response (201):**

```json
{
  "receipt_id": "rcpt_xyz789",
  "event_id": "evt_abc123",
  "status": "processed",
  "reason_code": "RCPT_ACCEPTED_PROCESSED"
}
```

**Error responses:**

| Status | Code | Description |
|--------|------|-------------|
| 401 | `RCPT_AUTH_INVALID` | Signature verification failed |
| 401 | `RCPT_AUTH_MISSING` | No signature header provided |
| 404 | `RCPT_EVENT_NOT_FOUND` | Referenced event does not exist |
| 400 | `RCPT_SCHEMA_INVALID` | Missing required fields |
| 409 | `RCPT_DUPLICATE_IGNORED` | Receipt already accepted (idempotent) |

### Query Receipts

```
GET /api/v1/receipts
```

Query receipts for your hooks. Supports filtering by event, status, and time range.

**Query parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `event_id` | string | Filter by specific event |
| `hook_id` | string | Filter by hook |
| `status` | string | Filter by receipt status |
| `from` | string | Start of time range (ISO 8601) |
| `to` | string | End of time range (ISO 8601) |

---

## Receipt Secret Management

### Get Signing Secret

```
GET /api/v1/hooks/:hookId/receipt-secret
```

Returns the current receipt signing secret for a hook.

**Response (200):**

```json
{
  "hook_id": "abc123def456",
  "receipt_signing_secret": "whsec_a1b2c3d4e5f6...",
  "created_at": "2026-02-08T12:00:00Z"
}
```

### Rotate Signing Secret

```
POST /api/v1/hooks/:hookId/receipt-secret/rotate
```

Generates a new signing secret. The old secret remains valid for 24 hours.

**Response (200):**

```json
{
  "hook_id": "abc123def456",
  "receipt_signing_secret": "whsec_new_secret...",
  "previous_secret_valid_until": "2026-02-09T12:00:00Z",
  "created_at": "2026-02-08T12:00:00Z"
}
```

---

## Replay

### Replay Single Event

```
POST /api/replay
```

Replays a captured webhook event. Requires Pro tier.

**Request body:**

```json
{
  "logId": "log_abc123",
  "targetUrl": "http://localhost:3000/webhook",
  "targetType": "dev",
  "signatureMode": "strip",
  "include_confirmed": false,
  "audit_note": null
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `logId` | string | Yes | The ID of the event to replay |
| `targetUrl` | string | No | Override target URL (defaults to hook URL) |
| `targetType` | string | No | `dev` (default), `staging`, `prod` |
| `signatureMode` | string | No | `strip` (default), `preserve_original`, `resign` |
| `include_confirmed` | boolean | No | Allow replaying confirmed events (default: false) |
| `audit_note` | string | Conditional | Required when `include_confirmed` is true |

**Response (200):**

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

**Error (403) -- Pro tier required:**

```json
{
  "error": "Replay is a Pro feature. Upgrade to access.",
  "code": "TIER_REQUIRED"
}
```

**Error (400) -- Event already confirmed:**

```json
{
  "error": "Event already confirmed. Set include_confirmed=true with audit_note to override.",
  "code": "ALREADY_CONFIRMED"
}
```

### List Replay Jobs

```
GET /api/replay?hookId=abc123&limit=50
```

Returns replay jobs for a hook.

### Cancel Replay Job

```
DELETE /api/replay?jobId=job_xyz789
```

Cancels a pending or blocked replay job.

### Batch Replay

```
POST /api/replay/batch
```

Batch replay with dry-run preview. Requires Pro tier.

**Request body:**

```json
{
  "hookId": "abc123",
  "filter": {
    "processing_status": ["applied_unknown", "applied_failed"],
    "time_range": {
      "from": "2026-02-01T00:00:00Z",
      "to": "2026-02-08T00:00:00Z"
    }
  },
  "dry_run": true
}
```

---

## Investigations

### Create Investigation

```
POST /api/investigations
```

**Request body:**

```json
{
  "hookId": "abc123",
  "triggerEventId": "event-uuid",
  "title": "Payment webhook failing",
  "autoLink": true
}
```

### List Investigations

```
GET /api/investigations
GET /api/investigations?hookId=abc123
GET /api/investigations?status=open,investigating
```

### Get Investigation

```
GET /api/investigations/:id
```

### Update Investigation

```
PATCH /api/investigations/:id
```

### Resolve Investigation

```
PATCH /api/investigations/:id
```

**Request body:**

```json
{
  "action": "resolve",
  "resolutionSummary": "API key was expired",
  "resolutionType": "config_fix"
}
```

---

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| All API endpoints | 10 requests per minute per IP |
| Test events | 5 per minute per user |
| Webhook ingress | 100 per minute per hook |

When rate-limited, responses include:

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1707393600
```

---

## Error Format

All error responses follow this format:

```json
{
  "error": "Human-readable error message"
}
```

Or for structured errors:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable description",
    "upgrade_url": "/pricing"
  }
}
```

Common error codes:

| Code | Status | Description |
|------|--------|-------------|
| `AUTH_REQUIRED` | 401 | No valid authentication provided |
| `PLAN_LIMIT` | 403 | Tier limit reached |
| `TIER_REQUIRED` | 403 | Feature requires a higher plan |
| `RATE_LIMITED` | 429 | Too many requests |
| `BACKEND_UNAVAILABLE` | 502/504 | Data plane is temporarily unreachable |
| `ALREADY_CONFIRMED` | 400 | Event already has a confirmed receipt |
| `PROD_REPLAY_BLOCKED` | 403 | Production replay safety gate triggered |

---

## What's Next?

- [Getting Started](/) -- Create your first webhook endpoint
- [Receipt Signing](/receipt-signing/) -- Authenticate API calls with HMAC-SHA256
- [CLI Tool](/features/cli/) -- Use the CLI for command-line access
