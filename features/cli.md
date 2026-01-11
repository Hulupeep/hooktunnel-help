---
layout: default
title: HookTunnel CLI
description: Forward webhooks to localhost for development
permalink: /features/cli/
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

## Use Cases

### Local Development with Live Webhooks

Test your webhook handler against real provider events:

```bash
# Terminal 1: Start your local server
npm run dev  # Listening on port 3000

# Terminal 2: Connect HookTunnel
hooktunnel connect dev 3000

# Now configure Stripe/Twilio/GitHub to send webhooks to:
# https://hooks.hooktunnel.com/h/<your-hook-id>
```

Every webhook from the provider forwards to your localhost in real-time.

---

### Debug Failed Webhooks

Find and replay failed requests to debug issues:

```bash
# 1. Check recent logs for errors
hooktunnel logs abc123 --limit 50

# Look for 500 status codes in the output

# 2. Fix your code locally

# 3. Replay the failed request to test your fix
hooktunnel replay log_xyz123 --to http://localhost:3000/webhook
```

---

### AI-Assisted Debugging with Claude Code

Use the CLI in Claude Code sessions for intelligent troubleshooting:

**Example prompt:**
> "Use hooktunnel to find the last 10 webhook errors and explain what's causing them"

Claude Code will:
```bash
# Fetch recent logs
hooktunnel logs abc123 --limit 10

# Analyze the 500 errors and explain:
# - What payload caused the failure
# - Why your handler returned an error
# - Suggested fixes
```

**Example prompt:**
> "Replay the failed Stripe webhook and show me what my server returns"

```bash
# Claude runs:
hooktunnel replay log_abc123 --to http://localhost:3000/webhook

# Then explains the response and any errors
```

**Example prompt:**
> "Monitor webhooks and alert me if any fail"

```bash
# Claude starts the tunnel in verbose mode
hooktunnel connect dev 3000 --verbose

# Watches the output and explains each request
# Alerts you when a 4xx/5xx status appears
```

---

### Team Development

Multiple developers can work with the same hook:

```bash
# Developer 1 (working on payment handler)
hooktunnel connect dev 3000

# Developer 2 (reviewing logs)
hooktunnel logs abc123

# Both see the same webhook events
```

---

### CI/CD Webhook Testing

Validate webhook handlers in your test pipeline:

```bash
#!/bin/bash
# ci-webhook-test.sh

# Start server in background
npm run start &
SERVER_PID=$!

# Connect tunnel
hooktunnel connect dev 3000 &
TUNNEL_PID=$!

# Wait for connections
sleep 5

# Run your webhook tests
npm run test:webhooks

# Cleanup
kill $SERVER_PID $TUNNEL_PID
```

---

### Switching Between Environments

Work with different webhook environments:

```bash
# Development
hooktunnel connect dev 3000

# Staging (different port)
hooktunnel connect staging 3001

# Production monitoring (read-only, no forwarding)
hooktunnel logs prod-hook-id --limit 100
```

---

### Debugging Signature Verification

When webhooks fail signature verification:

```bash
# 1. Enable verbose mode to see full request details
hooktunnel connect dev 3000 --verbose

# 2. Check the headers being sent
# The CLI shows all headers including signatures

# 3. Compare with what your handler expects
```

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
