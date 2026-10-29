# BounceBan Claude Code Plugin

A [Claude Code](https://claude.com/claude-code) plugin for verifying email addresses with the [BounceBan API](https://bounceban.com/public/doc/api.html). BounceBan specializes in accept-all (catch-all) emails and emails protected by Secure Email Gateways — it identifies which accept-all emails are actually deliverable instead of marking them all as risky.

Packaged from the [bounceban-com/skill-email-verification](https://github.com/bounceban-com/skill-email-verification) skill.

## What it does

Once installed, Claude can:

- **Verify a single email** — synchronously (waterfall endpoint, waits up to 80 s) or asynchronously (submit, then poll or receive a webhook).
- **Verify email lists in bulk** — up to ~500k emails per task via JSON, or a CSV file up to 25 MB; poll progress, fetch paginated results, export to a CSV download link, or stop a running task (credits for unverified emails are refunded).
- **Run quick checks** — free / disposable / role / syntax detection without an SMTP handshake (`GET /v1/check`).
- **Manage the account** — check credit balance and live rate limits (`GET /v1/account`).
- **Handle webhooks** — payload formats and event types for async notifications.

The skill encodes credit-saving rules (never resubmit while a verification is in flight, retry waterfall timeouts for free, polling caps) and a full HTTP error-handling table, so Claude doesn't waste credits or misread API responses.

## Setup

1. Get an API key at <https://bounceban.com/app/api/settings>.
2. Export it as an environment variable (e.g. in your shell profile):

   ```bash
   export BOUNCEBAN_API_KEY="your-api-key"
   ```

3. Make sure `curl` is available (it is on virtually every system).

The key is sent in the `Authorization` header **without** a `Bearer` prefix. Each verification costs 1 credit.

## Installation

### From a local checkout

```bash
git clone https://github.com/bounceban-com/claudecode-plugin-email-verification.git
claude --plugin-dir ./claudecode-plugin-email-verification
```

This loads the plugin for the session — useful for trying it out or for development. Run `/reload-plugins` inside Claude Code to pick up local changes.

### Via the community marketplace

Once the plugin is available in the [Claude Code community marketplace](https://github.com/anthropics/claude-plugins-community), install it with:

```shell
/plugin marketplace add anthropics/claude-plugins-community
/plugin install bounceban@claude-community
```

## Usage

Ask naturally — the skill triggers on email-verification tasks:

- *"verify jane@acme.com"*
- *"clean this email list"*
- *"is this a disposable address?"*
- *"how many BounceBan credits do I have left?"*

Or invoke the slash command directly:

```shell
/bounceban:verify jane@acme.com
```

## Structure

```
.claude-plugin/
  plugin.json                         # Plugin manifest
skills/
  email-verification/
    SKILL.md                          # Entry point: endpoint routing table, quick recipes, critical rules
    references/
      introduction.md                 # Auth, credits, result enums, rate limits
      single-verification.md          # Sync (waterfall) and async single verification
      bulk-verification.md            # Bulk tasks: create, status, dump, export, stop, destroy
      check.md                        # Fast free/disposable/role/syntax check (no SMTP)
      account.md                      # Credits balance and rate limits
      webhooks.md                     # Webhook payloads and event types
  verify/
    SKILL.md                          # /bounceban:verify <email...> slash command
```

Claude reads `SKILL.md` first and loads the matching reference file before calling an endpoint, keeping context usage small.

## Rate limits

| Endpoint | Limit |
| --- | --- |
| `/verify/single` | 25 req/s |
| `/verify/bulk` | 3 req/s |
| Others | 25 req/s |

Live values are returned by `GET /v1/account`.

## Links

- [BounceBan API documentation](https://bounceban.com/public/doc/api.html)
- [Get an API key](https://bounceban.com/app/api/settings)
- [Pricing](https://bounceban.com/pricing)
- [Claude Code plugin docs](https://code.claude.com/docs/en/plugins)

## License

[MIT](LICENSE)
