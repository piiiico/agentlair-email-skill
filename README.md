# AgentLair Email Skill

> Give your AI agent a real email address in 3 API calls. No SMTP. No IMAP. No CAPTCHA.

This is an [Agent Skill](https://agentskills.io) for [AgentLair](https://agentlair.dev) — an email infrastructure service built specifically for AI agents.

## Install

Add `SKILL.md` to your project or `.claude/skills/` directory, or reference it from your agent config:

```
https://raw.githubusercontent.com/piiiico/agentlair-email-skill/main/SKILL.md
```

## What it does

Teaches your AI agent how to:

1. **Get an API key** from AgentLair (instant, no signup, no CAPTCHA)
2. **Claim an email address** like `myagent@agentlair.dev`
3. **Send emails** via REST — DKIM/SPF/DMARC preconfigured
4. **Check inbox** and read messages

## Compatible platforms

Works with any platform supporting [agentskills.io](https://agentskills.io) format:

- Claude Code
- Cursor
- GitHub Copilot (VS Code)
- OpenAI Codex
- Gemini CLI
- Roo Code, Goose, OpenHands, and 25+ more

## Example (3 API calls)

```bash
# 1. Get an API key
curl -X POST https://agentlair.dev/v1/auth/keys

# 2. Claim an address
curl -X POST https://agentlair.dev/v1/email/claim \
  -H "Authorization: Bearer al_live_..." \
  -d '{"address": "myagent@agentlair.dev"}'

# 3. Send email
curl -X POST https://agentlair.dev/v1/email/send \
  -H "Authorization: Bearer al_live_..." \
  -d '{"from": "myagent@agentlair.dev", "to": ["user@example.com"], "subject": "Hello", "text": "Sent by an agent"}'
```

## Why AgentLair?

Every other email API (Resend, Mailgun, SendGrid) requires a **human** to sign up — CAPTCHA, phone verification, domain ownership, DNS configuration. Agents can't do any of that.

AgentLair is designed for agents: one HTTP request to get a key, one more to claim an address, done.

- ✅ No account required to start
- ✅ `@agentlair.dev` addresses ready to use (custom domains coming Q2 2026)
- ✅ DKIM/SPF/DMARC preconfigured
- ✅ HTTP 402 / x402 support: agents can pay for higher limits autonomously
- ✅ Free tier: 50 emails/day, 100 API calls/day

## Links

- 🌐 **Product**: [agentlair.dev](https://agentlair.dev)
- 📖 **API docs**: [agentlair.dev/docs](https://agentlair.dev/docs)
- 🤖 **MCP server**: [agentlair-mcp-server.amdal-dev.workers.dev/mcp](https://agentlair-mcp-server.amdal-dev.workers.dev/mcp)

## License

MIT
