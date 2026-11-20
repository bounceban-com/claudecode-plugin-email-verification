---
name: verify
description: Verify one or more email addresses right now with the BounceBan API and report the results.
---

# Verify email(s)

Verify the email address(es) the user provided: "$ARGUMENTS"

Follow the instructions and critical rules in the `email-verification` skill of this plugin (`skills/email-verification/SKILL.md`), reading its reference files as needed.

Procedure:

1. If `$ARGUMENTS` is empty, ask the user which email address(es) to verify.
2. Confirm `$BOUNCEBAN_API_KEY` is set (`[ -n "$BOUNCEBAN_API_KEY" ]`). If not, tell the user to get a key at https://bounceban.com/app/api/settings and export it as `BOUNCEBAN_API_KEY`, then stop.
3. For 1–5 emails: use the synchronous waterfall endpoint for each (it holds the connection up to 80 s):

   ```bash
   curl -s "https://api-waterfall.bounceban.com/v1/verify/single?email=ADDRESS" \
     -H "Authorization: $BOUNCEBAN_API_KEY"
   ```

   On HTTP 408, retry the same request (free within 30 minutes).
4. For more than 5 emails (or a file): create a bulk task with `POST /v1/verify/bulk` (or `/v1/verify/bulk/file` for CSV), poll `/v1/verify/bulk/status`, then fetch results from `/v1/verify/bulk/dump`. If the user asks to cancel a running task, stop it with `POST /v1/verify/bulk/stop` (credits for unverified emails are refunded) and report `not_verified_count` alongside the result counts.
5. Report per email: `result` (deliverable / risky / undeliverable / unknown), `score`, and notable flags (`is_disposable`, `is_accept_all`, `is_role`, `is_free`). For lists, summarize counts by result and list the undeliverable addresses.

Never resubmit an email while its status is `verifying` or `queue` — each submit costs 1 credit.
