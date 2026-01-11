---
layout: default
title: HookTunnel CLI
description: Forward webhooks to localhost for development
---

# HookTunnel CLI

Forward webhooks from HookTunnel to your local development server.

---

## Installation

```bash
# Using npx (no installation required)
npx hooktunnel-cli

# Or install globally
npm install -g hooktunnel-cli
```

**Requirements:** Node.js 18+

---

## Quick Start

```bash
# 1. Login with your API key
hooktunnel login --key <your-api-key>

# 2. Start forwarding webhooks to localhost:3000
hooktunnel connect dev 3000

# 3. Send webhooks to your HookTunnel URL
# They'll appear in your terminal and forward to localhost
```

---

## Commands

### `hooktunnel login`

Authenticate with your API key.

```bash
# Login with API key
hooktunnel login --key ht_abc123...

# Get your API key from:
# https://hooktunnel.com/app/settings
```

### `hooktunnel connect <env> <port>`

Connect to HookTunnel and forward webhooks to localhost.

```bash
# Forward to localhost:3000
hooktunnel connect dev 3000

# With custom host
hooktunnel connect dev 3000 --host 127.0.0.1

# Verbose mode for debugging
hooktunnel connect dev 3000 --verbose
```

**Environments:**
- `dev` - Development environment
- `staging` - Staging environment
- `prod` - Production environment

**Output:**
```
🔗 HookTunnel
  Environment: dev
  Local port: 3000

✓ Connected to HookTunnel
  Session: abc12345...
  Forwarding to: http://localhost:3000

Waiting for webhooks... (Ctrl+C to stop)

[12:00:01] POST   /webhook    200  (45ms)
[12:00:05] POST   /webhook    500  (12ms)
```

### `hooktunnel hooks`

List your webhook endpoints.

```bash
hooktunnel hooks
```

**Output:**
```
📌 Your Hooks (2)

  ID                      Provider    Status    Requests
  ------------------------------------------------------------
  abc123def456ghi789      stripe      active    142
  xyz789abc123def456      twilio      active    57

  Webhook URL: https://hooks.hooktunnel.com/h/<hook_id>
```

### `hooktunnel logs <hookId>`

View recent request logs for a hook.

```bash
# View last 20 logs
hooktunnel logs abc123def456

# View last 50 logs
hooktunnel logs abc123def456 --limit 50
```

**Output:**
```
📋 Request Logs for abc123def456... (20)

  Time                Method  Path                          Status  Size
  ---------------------------------------------------------------------------
  1/11/2026 12:00:05  POST    /webhook                      200     1.2KB
  1/11/2026 11:58:32  POST    /webhook                      500     0.8KB
  ...

  Log ID (for replay): log_abc123...
```

### `hooktunnel replay <logId>` (Pro)

Replay a captured request.

```bash
# Replay to your connected tunnel
hooktunnel replay log_abc123

# Replay to specific URL
hooktunnel replay log_abc123 --to http://localhost:4000/webhook
```

**Note:** Replay requires Pro tier.

### `hooktunnel status`

Show current status and configuration.

```bash
hooktunnel status
```

**Output:**
```
📊 HookTunnel Status

  Authentication: ✓ Logged in
  API Key: ht_abc1...
  Config: /home/user/.config/hooktunnel-cli/config.json

  Hooks: 2
  Active: 2

  Recent hooks:
    - abc123def456... (active)
    - xyz789abc123... (active)
```

### `hooktunnel logout`

Clear stored credentials.

```bash
hooktunnel logout
```

---

## Troubleshooting

### Connection Failed

```
Error: Failed to connect to HookTunnel
```

**Solutions:**
1. Check your internet connection
2. Verify your API key is valid: `hooktunnel login --key <key>`
3. Try with verbose mode: `hooktunnel connect dev 3000 --verbose`

### Authentication Required

```
Error: Authentication required
Suggestion: Run "hooktunnel login" to authenticate
```

**Solution:** Login with your API key first.

### Local Server Not Responding

```
[12:00:01] POST   /webhook    502  (5ms)
```

**Solutions:**
1. Make sure your local server is running
2. Check the port number matches your server
3. Verify your server is listening on localhost

### Pro Feature Required

```
Error: Pro tier required
Suggestion: Upgrade at https://hooktunnel.com/#pricing
```

**Solution:** Upgrade to Pro for replay and other advanced features.

---

## Configuration

The CLI stores configuration in:
- **Linux/macOS:** `~/.config/hooktunnel-cli/config.json`
- **Windows:** `%APPDATA%/hooktunnel-cli/config.json`

---

## Getting an API Key

1. Go to [hooktunnel.com/app/settings](https://hooktunnel.com/app/settings)
2. Click "Generate API Key"
3. Copy the key (starts with `ht_`)
4. Run `hooktunnel login --key <your-key>`

---

## See Also

- [Getting Started](/getting-started/)
- [Smart Search](/features/semantic-search/)
- [Investigations](/features/investigations/)
