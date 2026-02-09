---
layout: default
title: Smart Investigations
description: Track and resolve webhook issues with AI-powered pattern matching and anomaly detection
permalink: /features/investigations/
---

# Smart Investigations

Turn mysterious webhook errors into tracked, resolved issues. Investigations help you group related events, identify patterns, and build institutional knowledge about recurring problems.

---

## Quick Start

1. Find a problematic webhook event in your event list
2. Click the **Investigate** button (magnifying glass icon)
3. HookTunnel automatically links similar events
4. Add notes as you debug
5. Mark resolved when fixed

---

## Creating an Investigation

### From Event List
Click the investigate icon next to any event. HookTunnel will:
- Create a new investigation with the event as the trigger
- Auto-link similar events from the same time window
- Suggest a possible cause if a pattern is recognized
- Calculate an anomaly score

### From Live Monitor
Switch to Live view mode in your hook dashboard, then click any event to investigate.

---

## Investigation Features

### Auto-Linking
When you start an investigation, HookTunnel automatically finds related events:
- **Similar requests** - Same method, path, and error signature
- **Time window** - Events within 5 minutes of the trigger
- **Pattern match** - Events matching known error patterns

### AI-Suggested Causes
If HookTunnel has seen this pattern before, it suggests what caused it and how it was resolved:

```
Suggested Cause: API rate limit exceeded
Resolution: Implemented exponential backoff
Confidence: 87%
```

This comes from pattern memory - every resolved investigation teaches HookTunnel about your system.

### Anomaly Scoring
Each investigation gets a severity score based on:
- **Error rate deviation** - How unusual is this error rate?
- **Volume changes** - Traffic spike or drop?
- **Scope** - How many events are affected?
- **Status codes** - Unusual response codes?

| Score | Verdict | Meaning |
|-------|---------|---------|
| 0-0.25 | Noise | Normal variation, likely false alarm |
| 0.25-0.5 | Notable | Worth checking but not urgent |
| 0.5-0.75 | Incident | Significant deviation, needs attention |
| 0.75-1.0 | Critical | Major anomaly, investigate immediately |

---

## Investigation Workflow

### Status States
- **Open** - Just created, needs triage
- **Investigating** - Actively debugging
- **Resolved** - Root cause found and fixed
- **Archived** - Closed without resolution

### Adding Notes
Document your debugging process:
```
14:23 - Checked Stripe dashboard, no issues on their side
14:25 - Found rate limit error in logs
14:30 - API key was misconfigured in staging
```

### Resolution Types
When resolving, categorize the fix:
- **Config Fix** - Environment, settings, or config change
- **Code Fix** - Bug fix in application code
- **External Issue** - Provider or third-party problem
- **False Alarm** - Not actually an issue
- **Workaround** - Temporary fix applied
- **Other** - Doesn't fit categories

---

## Investigations List

Access all your investigations at `/app/investigations`:

- **Filter by status** - Open, Investigating, Resolved, Archived
- **View stats** - Open count, in-progress, resolved, high priority
- **Quick navigation** - Click to jump to hook detail with investigation panel

---

## Live Event Monitor

Watch events stream in real-time:

1. Go to any hook's detail page
2. Click the **Live** view mode button
3. Events appear as they arrive (3-second polling)
4. Click any event to investigate or view details

Features:
- **Pause/Resume** - Stop auto-refresh when analyzing
- **New event highlighting** - Fresh events flash blue
- **Auto-scroll** - Stays at top when live

---

## Cross-Customer Pattern Detection

When an issue affects multiple teams simultaneously, HookTunnel flags it as a **Widespread Issue** — suggesting an external outage rather than a problem with your specific configuration.

### How It Works
1. When you open an investigation with a notable anomaly score (0.25+), HookTunnel checks if other teams are seeing the same pattern
2. If 3 or more teams are affected within 24 hours, a **Widespread Issue** badge appears
3. If a known provider is involved, a link to their status page is shown

### What You See
- **Purple "Widespread Issue" badge** in the investigation header
- **Alert panel** showing how many other teams are affected
- **Provider status link** (Stripe, Twilio, GitHub, etc.) when applicable

### Privacy Guarantees
- You only see aggregate counts ("5 other teams affected")
- No other customer's data, hook IDs, or identifiers are ever exposed
- Pattern matching uses structural fingerprints, not payload data

### Supported Providers
| Provider | Status Page |
|----------|-------------|
| Stripe | status.stripe.com |
| Twilio | status.twilio.com |
| GitHub | githubstatus.com |
| SendGrid | status.sendgrid.com |
| Shopify | shopifystatus.com |

---

## Pattern Memory

HookTunnel learns from your resolved investigations:

### How It Works
1. You resolve an investigation with a description
2. HookTunnel stores the pattern fingerprint
3. Future similar events trigger a suggestion
4. Confidence increases with repeated patterns

### What's Stored
- Error signature (status codes, paths)
- Temporal pattern (burst, gradual, periodic)
- Resolution summary and type
- Average time to resolve
- Success rate

### Privacy
Pattern memory is anonymized - no payload data is stored, only structural patterns.

---

## Best Practices

1. **Investigate early** - Start tracking before you forget context
2. **Add notes** - Future you will thank present you
3. **Resolve properly** - Good resolutions train better suggestions
4. **Use categories** - Helps identify systemic issues
5. **Check anomaly scores** - Prioritize high-severity investigations

---

## API Reference

### Create Investigation
```bash
POST /api/investigations
{
  "hookId": "abc123",
  "triggerEventId": "event-uuid",
  "title": "Payment webhook failing",
  "autoLink": true
}
```

### List Investigations
```bash
GET /api/investigations
GET /api/investigations?hookId=abc123
GET /api/investigations?status=open,investigating
```

### Get Investigation Details
```bash
GET /api/investigations/{id}
```

### Update Investigation
```bash
PATCH /api/investigations/{id}
{
  "status": "investigating"
}
```

### Resolve Investigation
```bash
PATCH /api/investigations/{id}
{
  "action": "resolve",
  "resolutionSummary": "API key was expired",
  "resolutionType": "config_fix"
}
```

### Check Widespread Patterns
```bash
GET /api/investigations/widespread
GET /api/investigations/widespread?hookId=abc123&patternHash=xyz
```

---

## FAQ

**Q: Can other customers see my investigation details?**
No. Cross-customer detection only shares aggregate counts (e.g., "5 teams affected"). Your events, hooks, and investigation notes are never visible to anyone else.

**Q: What triggers a "Widespread Issue" badge?**
When 3 or more distinct teams have investigations with similar anomaly patterns within the last 24 hours.

**Q: Does HookTunnel check provider status pages automatically?**
Not yet — currently HookTunnel links to provider status pages for manual checking. Automated polling is planned for a future release.

**Q: How accurate are AI-suggested causes?**
Accuracy improves over time as more investigations are resolved. The confidence percentage reflects how closely the current pattern matches known patterns. Treat suggestions below 70% as hints rather than answers.

**Q: How long is pattern memory retained?**
Pattern memory is stored indefinitely and improves with each resolved investigation. Older patterns with low success rates are ranked lower.
