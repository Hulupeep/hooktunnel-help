---
layout: default
title: Smart Search - Find Webhooks by Meaning
description: Use semantic search to find webhook requests by intent, not just exact text. Discover similar requests and debug patterns faster.
permalink: /features/semantic-search/
---

# Smart Search: Find Webhooks by Meaning

**Stop scrolling. Start searching.**

HookTunnel's Smart Search uses AI-powered semantic search to help you find webhook requests by what they *mean*, not just what they contain. It's like having a search engine that understands webhooks.

---

## How It Compares

| Feature | HookTunnel | RequestBin | Webhook.site | ngrok |
|---------|------------|------------|--------------|-------|
| Semantic search | Yes | No | No | No |
| Find similar requests | Yes | No | No | No |
| Search by intent | Yes | No | No | No |
| Offline search | Yes | No | No | No |
| Filter by text | Yes | Yes | Yes | Yes |

Most webhook tools only offer basic text filtering. HookTunnel understands the *meaning* behind your search.

---

## What Is Smart Search?

Smart Search converts your webhook requests into mathematical representations (vectors) that capture their characteristics:

- **HTTP method** (GET, POST, PUT, DELETE)
- **URL path patterns** (/checkout, /webhooks/stripe, /api/users)
- **Provider fingerprints** (Stripe, Twilio, GitHub, etc.)
- **Header patterns** (content types, authentication headers)
- **Payload characteristics** (size, structure hints)
- **Timing patterns** (when requests arrived)

This lets you search by *concept* rather than exact text matching.

---

## Getting Started

### Accessing Smart Search

1. Navigate to any webhook's detail page (`/app/hooks/[your-hook-id]`)
2. Click the **Search** button in the toolbar (next to List)
3. Type your search query in natural language

### Basic Search Examples

| You Type | What It Finds |
|----------|---------------|
| `stripe payment` | Requests that look like Stripe payment webhooks |
| `POST checkout` | POST requests to checkout-related paths |
| `failed 500` | Requests that resulted in 500 errors |
| `twilio sms` | Twilio SMS-related webhook events |
| `large payload` | Requests with significant body sizes |

---

## Finding Similar Requests

One of Smart Search's most powerful features is **Find Similar** - it shows you requests that share patterns with a selected request.

### How to Use Find Similar

1. Click on any request in your event list
2. Click the **Similar** button in the toolbar (or in the detail modal)
3. The sidebar shows requests ranked by similarity percentage

### Understanding Similarity Scores

| Score | Meaning |
|-------|---------|
| 90-100% | Nearly identical pattern (same method, path, provider) |
| 70-89% | Very similar (same type of request, minor differences) |
| 50-69% | Related (shares some characteristics) |
| Below 50% | Weak match (few shared patterns) |

---

## Real-World Scenarios

### Scenario 1: Debugging a Failed Payment

**The problem:** A customer reports a failed checkout. You have thousands of webhook events.

**Without Smart Search:**
```
*scrolls through 500 events*
*Ctrl+F "checkout"*
*finds 200 matches*
*scrolls more*
"Which one was it again?"
```

**With Smart Search:**
1. Type `stripe checkout failed` in search
2. Results show payment-related webhooks ranked by relevance
3. Click the suspicious one, then **Find Similar**
4. Discover 3 other failures with the same pattern
5. Realize they all came from the same customer IP

**Time saved:** 20 minutes → 30 seconds

---

### Scenario 2: Finding a Request You Saw Last Week

**The problem:** You remember seeing a weird webhook last Tuesday. Something about user signups.

**Without Smart Search:**
```
*filters by date*
*scrolls through 300 events*
*gives up*
"I'll just check the logs"
```

**With Smart Search:**
1. Type `user signup POST`
2. Results show user registration webhooks
3. Spot the anomaly immediately - one has an unusual payload size

---

### Scenario 3: Grouping Related Events

**The problem:** You want to see all webhook events related to a specific flow (e.g., a complete checkout process: cart → payment → confirmation).

**With Find Similar:**
1. Find one request from the flow
2. Click **Find Similar**
3. See all related requests clustered together
4. Understand the full webhook sequence

---

## Tips for Better Searches

### Be Descriptive, Not Exact

Smart Search understands intent. Instead of searching for the exact path:

| Instead of | Try |
|------------|-----|
| `/api/v1/webhooks/stripe/checkout` | `stripe checkout` |
| `POST /users/register` | `user registration POST` |
| `application/json` | `json payload` |

### Combine Concepts

Mix different aspects of what you're looking for:

- `stripe payment large` - Stripe payments with big payloads
- `POST error today` - Failed POST requests from today
- `twilio voice callback` - Twilio voice-related callbacks

### Use Find Similar for Patterns

When debugging:
1. Find one example of the problem
2. Use **Find Similar** to find all related cases
3. Look for patterns in the similarity cluster

---

## How It Works (Technical)

Smart Search runs entirely in your browser using WebAssembly (WASM). Here's what happens:

1. **Indexing**: When you load a webhook page, requests are converted into 384-dimensional vectors
2. **Storage**: The index is saved in your browser's IndexedDB for offline access
3. **Search**: Your query is converted to a vector and compared using cosine similarity
4. **Results**: Requests are ranked by how similar their vectors are to your query

### Performance

- **Initialization**: < 100ms
- **Search time**: < 50ms for 10,000 requests
- **Index size**: ~1KB per request

### Privacy

All search processing happens in your browser. Your webhook data never leaves your machine for search purposes.

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `/` | Focus search input |
| `Escape` | Clear search / close modal |
| `Enter` | Select first result |
| `↑` `↓` | Navigate results |

---

## Troubleshooting

### "Search is initializing..."

The WASM engine is loading. This typically takes less than a second on first load.

### "No matching requests found"

Try broader search terms. Smart Search finds conceptual matches, but very specific queries might not match if your requests don't share those patterns.

### Search feels slow

If you have 10,000+ requests, initial indexing may take a moment. Subsequent searches will be fast as the index is cached.

---

## What's Next?

- **Saved Searches**: Save common queries for quick access
- **Search Alerts**: Get notified when new requests match a saved search
- **Cross-Hook Search**: Search across all your webhooks at once

---

## Feedback

Found a bug? Have a feature request? [Open an issue](https://github.com/Hulupeep/hooktunnel-web/issues) or reach out to support.
