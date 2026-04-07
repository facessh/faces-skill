# Billing & Credits

## Check balance

```bash
faces billing:balance --json
```

Returns `credits_remaining`, `is_active`, and plan type.

## Top up credits

```bash
faces billing:topup --amount 5
```

Minimum $1. Requires a saved payment method. Credits are applied immediately.

## Add a payment method

```bash
faces billing:card-setup
```

Opens a Stripe Checkout session to save a card. Required before `billing:topup`
can charge.

## Check subscription

```bash
faces billing:subscription --json
```

Returns plan type (`free` or `connect`), renewal date, and included token
allowance.

## Rough cost reference

| Operation | Approximate cost |
|-----------|-----------------|
| Audio/video transcription | ~$0.37/hour |
| Text document compilation | Minimal (pennies for typical documents) |
| Chat message | Varies by model — see `faces models:list` for per-token pricing |

## Common scenarios

**402 during compilation:** Balance is empty. Run `faces billing:balance --json`
to confirm, then `faces billing:topup --amount <USD>` to add credits and retry.

**Connect plan users:** The Connect plan ($17/month) includes 100k compile
tokens. Check remaining allowance with `faces billing:subscription --json`.
Chat via ChatGPT passthrough (GPT models) is free on Connect — no credit
cost for chat messages routed through your ChatGPT subscription.
