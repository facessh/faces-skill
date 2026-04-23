# Authentication & Registration

## Registering a new account

Registration requires a credit card payment to activate. The flow is:

### Step 1 — Create the account

Ask the user which plan they want, then register with `--plan`:

```bash
# Pay-per-token plan ($5 activation, 5% markup on all API calls)
faces auth:register \
  --email user@example.com \
  --password 'SecurePass123!' \
  --username alice \
  --json

# Subscription Connect plan ($17/month, link your ChatGPT account for select OpenAI models)
faces auth:register \
  --email user@example.com \
  --password 'SecurePass123!' \
  --username alice \
  --plan connect \
  --json
```

`--plan` defaults to `free` (pay-per-token) if omitted.

The response includes:

```json
{
  "status": "registered",
  "user_id": "...",
  "activation_required": true,
  "activation_checkout_url": "https://checkout.stripe.com/c/pay/..."
}
```

The JWT is saved automatically. The account exists but is not yet active.

### Step 2 — Activate via payment

The `activation_checkout_url` is a Stripe Checkout page. The human must open it in a browser and complete payment.

Tell the user:

> Paste this link into your browser and complete the payment. When you see the confirmation page, come back here and let me know it's done.
>
> `<activation_checkout_url>`

Do **not** attempt to open the URL or complete payment programmatically. Only a human can do this step.

### Step 3 — Confirm activation

Once the user says payment is done:

```bash
faces billing:balance --json
```

Check that `is_active` is `true`. If not, tell the user the payment may not have gone through and ask them to try the link again.

### Full example

```bash
# 1. Register (use --plan connect for Subscription Connect plan)
RESULT=$(faces auth:register --email user@example.com --password 'SecurePass123!' --username alice --json)
URL=$(echo "$RESULT" | jq -r '.activation_checkout_url')

# 2. Give URL to human, wait for them to confirm payment

# 3. Verify
faces billing:balance --json | jq '.is_active'
```

## Logging in

For existing accounts:

```bash
faces auth:login --email user@example.com --password 'SecurePass123!'
```

The JWT is saved to `~/.faces/config.json` automatically.

To verify:

```bash
faces auth:whoami
```

## API key authentication

For non-interactive use (scripts, deployed agents), API keys are preferred over JWT:

```bash
# Create a key (requires JWT)
faces keys:create --name "my-agent" --budget 10.00 --expires-days 30

# Use it
faces config:set api_key sk-proj-...
# Or set the environment variable:
export FACES_API_KEY=sk-proj-...
```

API keys support optional restrictions:
- `--face ALIAS` — restrict to specific faces (repeatable)
- `--model NAME` — restrict to specific LLM models (repeatable)
- `--budget F` — spending cap in USD
- `--expires-days N` — auto-expiry

## Plans

### Pay-per-token

- **Activation:** $5 minimum initial spend (added as API credits).
- **Pricing:** 5% markup on all API calls — both compilation (building faces) and inference (chatting with faces). Charged per token at the underlying provider's rate plus the markup.
- **No monthly fee.**

### Subscription Connect ($17/month)

- **Same pricing** as pay-per-token (5% markup on all API calls), plus:
- **ChatGPT passthrough:** Link your ChatGPT Plus or Pro account via `faces auth:connect openai` to use select OpenAI models at no additional cost (no markup, no per-token charge from Faces) when both compiling and chatting with faces. See [OAUTH.md](OAUTH.md) for supported models.
- **Fallback:** By default, OAuth failures return 422 errors (no silent fallback to paid keys). Enable fallback with `faces account:preferences api_fallback true`.

### Checking the current plan

```bash
faces billing:subscription --json   # plan, status, period end
faces billing:balance --json        # credit balance, is_active
faces billing:quota --json          # compile token usage
```

### Upgrading

```bash
faces billing:checkout --plan connect
```

Returns a Stripe Checkout URL. The human must open it in a browser to complete payment.

## JWT vs API key

| Command group | Requires |
|---|---|
| `faces auth:*`, `faces keys:*` | JWT only — run `faces auth:login` first |
| Everything else | JWT **or** API key |

## Credential storage

Credentials are stored in `~/.faces/config.json`. The CLI reads from there automatically, or from environment variables:

| Variable | Purpose |
|---|---|
| `FACES_TOKEN` | JWT authentication token |
| `FACES_API_KEY` | API key authentication |

## Account preferences

Server-side settings stored on the user's account:

```bash
faces account:preferences                            # view current
faces account:preferences default_model gpt-5.4      # set default model for new faces
faces account:preferences api_fallback true           # allow paid fallback when OAuth fails
```

| Key | Values | Default | Effect |
|---|---|---|---|
| `api_fallback` | `true` / `false` | `false` | When false, OAuth failures return 422 instead of falling back to paid system keys |
| `default_model` | any valid model | `gpt-5.4` | Model inherited by new faces that don't specify `--default-model` |

Note: these are server-side preferences (not local config). They persist across devices and sessions.

## Refreshing a token

If a JWT expires:

```bash
faces auth:refresh    # refresh existing token
faces auth:login ...  # or re-login
```
