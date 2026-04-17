# Billing & Credits

All billing commands: `faces billing:COMMAND`. Use `--json` on any command for
structured output.

## Check balance

```bash
faces billing:balance --json
```

Returns credit balance and payment method status.

## Top up credits

```bash
faces billing:topup --amount 5
```

Minimum $1. Requires a saved payment method. Credits are applied immediately.

## Add a payment method

```bash
faces billing:card-setup
```

Opens a Stripe session to save a card. Required before `billing:topup` can
charge.

## Check subscription

```bash
faces billing:subscription --json
```

Returns plan type (`free` or `connect`), face count, and renewal date.

## Upgrade plan

```bash
faces billing:checkout --plan connect
```

Opens a Stripe Checkout URL to upgrade to the Connect plan.

## Check compile token quota

```bash
faces billing:quota --json
```

Shows compile token quota and per-face stats. Connect plan users compile via OAuth (free, unlimited) — this quota is still tracked but is less relevant for Connect users.

## View LLM pricing

```bash
faces billing:llm-costs --json
faces billing:llm-costs --provider openai
faces billing:llm-costs --provider anthropic
```

Lists available LLMs and their per-token costs. Filter by provider.

## Usage analytics

```bash
faces billing:usage --json
faces billing:usage --group-by model --from 2026-03-01 --to 2026-03-31
```

Aggregated usage analytics. Group by `api_key`, `model`, `llm`, or `date`.

## Rough cost reference

| Operation | Approximate cost |
|-----------|-----------------|
| Audio/video transcription | ~$0.37/hour |
| Text document compilation | Minimal (pennies for typical documents) |
| Chat message | Varies by model — see `faces billing:llm-costs` for per-token pricing |

## Common scenarios

**402 during compilation:** Balance is empty. Run `faces billing:balance --json`
to confirm, then `faces billing:topup --amount <USD>` to add credits and retry.

**422 oauth_rejected:** OAuth request failed and fallback is disabled. Enable
fallback: `faces account:preferences api_fallback true`. If no credits, run
`faces billing:topup` first. See [OAUTH.md](OAUTH.md) for details.

**Connect plan users:** The Connect plan ($17/month) compiles via OAuth (free,
unlimited). Chat via ChatGPT passthrough (GPT models) is also free on Connect —
no credit cost for requests routed through your ChatGPT subscription. If OAuth
is not linked, compile and chat will fail with 422 until you either link OpenAI
(`faces auth:connect openai`) or enable paid fallback
(`faces account:preferences api_fallback true`).
