# OAuth — Connect ChatGPT (Subscription Connect plan only)

Subscription Connect users can link their own ChatGPT Plus/Pro account to use select OpenAI models at no extra cost for both compiling and chatting with faces.

**The human must approve once in their browser. After that, tokens are stored server-side and the agent never asks again.**

```bash
# 1. Check if already connected — do this first, skip below if openai is listed
faces auth:connections --json

# 2. If not connected — start the flow (opens browser automatically, or use --manual on headless)
faces auth:connect openai
# → prints URL, waits, detects completion automatically via polling

# 3. Confirm
faces auth:connections --json   # should show openai

# 4. Disconnect
faces auth:disconnect openai
```

**`--manual` flag** (headless / no browser on this machine): prints the authorize URL for the human to open on any device, polls for completion, also accepts a pasted callback URL as fallback.

**Tasking the human:** if `auth:connections` returns `[]`, tell the human: *"Run `faces auth:connect openai` in your terminal, or open the authorize URL in your browser and approve access. I'll detect it automatically."*

Once connected, OAuth routing is transparent — no flag needed on inference commands. Requests to gpt-5.x models route through the user's ChatGPT subscription automatically.

## Fallback behavior (Subscription Connect only)

All OAuth and fallback settings (`api_fallback`, `--oauth-only`) apply only to Subscription Connect users with a linked ChatGPT account. Pay-per-token users do not route through OAuth and are unaffected.

By default, `api_fallback` is `false` — if OAuth fails (token revoked, model unsupported, rate limit), the request returns a **422** error instead of silently falling back to paid system keys.

The 422 response includes:
- `error: "oauth_rejected"` — stable code
- `fallback_available: true/false` — whether the user has credit balance to fall back on
- `detail` — human-readable explanation

**To enable paid fallback:**
```bash
faces account:preferences api_fallback true
```

**Per-request override:**
```bash
faces chat:chat alice -m "hello" --oauth-only    # force OAuth, no fallback (even if api_fallback is true)
```

The `--oauth-only` flag is available on `chat:chat`, `chat:messages`, `chat:responses`, `compile:thread:create`, and `compile:thread:message`.

## Subscription Connect onboarding

A new Subscription Connect user who hasn't linked OAuth will get 422 errors when using select OpenAI models until they either:
1. Link their OpenAI account: `faces auth:connect openai`
2. Enable paid fallback: `faces account:preferences api_fallback true`
3. Pass `--oauth-only` explicitly (to diagnose which requests use OAuth)

The recommended onboarding path is to link OAuth first, then compile.

## Supported OAuth models

Current models supported via ChatGPT OAuth: `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex`, `gpt-5.2`.
