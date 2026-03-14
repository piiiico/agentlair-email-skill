---
name: agentlair-email
description: >
  Give an AI agent a real email address via AgentLair REST API.
  Covers: getting an API key, claiming an @agentlair.dev address, sending emails,
  checking inbox, and reading messages — all via REST, no SMTP or IMAP required.
  Use when an agent needs to send or receive email, check a mailbox, or establish
  an email identity. Trigger on: agent email, send email, check inbox, claim email,
  agentlair.
license: MIT
compatibility: Requires internet access. Bash or any HTTP client (curl, fetch, axios). No system packages needed.
metadata:
  author: pico@amdal.dev
  version: "1.0.0"
  product: AgentLair (agentlair.dev)
---

# AgentLair Email Skill

AgentLair gives AI agents a real email identity via a simple REST API.
No SMTP, no IMAP, no CAPTCHA, no email required to sign up.

**Base URL:** `https://agentlair.dev`
**Auth:** `Authorization: Bearer $AGENTLAIR_API_KEY`

---

## Step 1: Get an API key (self-service, instant)

```bash
curl -s -X POST https://agentlair.dev/v1/auth/keys \
  -H "Content-Type: application/json" \
  -d '{}'
```

Response:
```json
{"key": "al_live_...", "tier": "free"}
```

Store as `AGENTLAIR_API_KEY`. **Free tier:** 50 emails/day, 100 API calls/day.

---

## Step 2: Claim an email address

```bash
curl -s -X POST https://agentlair.dev/v1/email/claim \
  -H "Authorization: Bearer $AGENTLAIR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"address": "myagent@agentlair.dev"}'
```

Response:
```json
{"address": "myagent@agentlair.dev", "claimed": true, "already_owned": false}
```

Choose any available `<name>@agentlair.dev` address. Addresses are persistent.

---

## Step 3: Send an email

```bash
curl -s -X POST https://agentlair.dev/v1/email/send \
  -H "Authorization: Bearer $AGENTLAIR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "from": "myagent@agentlair.dev",
    "to": ["recipient@example.com"],
    "subject": "Hello from my agent",
    "text": "This email was sent by an AI agent."
  }'
```

Response:
```json
{"id": "out_...", "status": "sent", "sent_at": "...", "rate_limit": {"daily_remaining": 49}}
```

**Important:** Use `text` (not `body`) for the message content. Optional: `"html"` for HTML version, `"cc"` (array).

Emails are delivered via Amazon SES with DKIM/SPF/DMARC. External delivery: ~1-2 seconds.

---

## Step 4: Check inbox

```bash
curl -s "https://agentlair.dev/v1/email/inbox?address=myagent@agentlair.dev&limit=10" \
  -H "Authorization: Bearer $AGENTLAIR_API_KEY"
```

Response:
```json
{
  "messages": [
    {
      "message_id": "<abc123@eu-west-1.amazonses.com>",
      "from": "sender@example.com",
      "subject": "Re: Hello",
      "received_at": "2026-03-13T10:00:00Z",
      "read": false
    }
  ],
  "count": 1
}
```

**Note:** `message_id` includes RFC 2822 angle brackets `<...>`. Strip them before using in the read endpoint.

Inbox indexing for inbound emails: allow up to 120 seconds after delivery before polling.

---

## Step 5: Read a specific message

Strip `<>` from `message_id`, URL-encode, then fetch:

```bash
# message_id from inbox: <abc123@eu-west-1.amazonses.com>
MSG_ID="abc123%40eu-west-1.amazonses.com"
curl -s "https://agentlair.dev/v1/email/messages/$MSG_ID?address=myagent@agentlair.dev" \
  -H "Authorization: Bearer $AGENTLAIR_API_KEY"
```

In JavaScript/TypeScript:
```typescript
const rawId = message.message_id.replace(/^<|>$/g, "");
const encodedId = encodeURIComponent(rawId);
const url = `https://agentlair.dev/v1/email/messages/${encodedId}?address=${encodeURIComponent(address)}`;
```

---

## Check sent outbox

```bash
curl -s "https://agentlair.dev/v1/email/outbox?limit=5" \
  -H "Authorization: Bearer $AGENTLAIR_API_KEY"
```

---

## Common patterns

### Agent that sends a confirmation email
1. Claim `confirm-agent@agentlair.dev`
2. Send with `text` field containing the confirmation
3. Check inbox for replies after 120s

### Agent with persistent email identity
1. Claim address once, store API key + address in agent config
2. Check inbox on each relevant task (not polling — check on trigger)

### Multi-agent email routing
- Each agent gets its own address: `billing@agentlair.dev`, `support@agentlair.dev`, etc.
- Each address is owned by the same API key — inbox is per-address

---

## Free tier limits
- 50 outbound emails/day
- 100 API requests/day
- Multiple @agentlair.dev addresses per key

Rate limits reset daily at midnight UTC.

---

## Notes
- **Custom domains** coming Q2 2026
- **No authentication** needed for inbound — any email sent to `*@agentlair.dev` arrives in inbox
- **Docs:** https://agentlair.dev/docs
- **API key is the account** — no username/password. Keep it secret.
