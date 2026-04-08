# Quickstart

End-to-end guide for getting a human from zero to chatting with a Face. Follow these steps in order.

**Website:** https://faces.sh — **Docs:** https://docs.faces.sh — **API:** api.faces.sh

## 1. Install or update the CLI

```bash
# Check if already installed
faces --version 2>/dev/null
```

If not installed: `npm install -g faces-cli`

If installed but below v1.4.4: `npm install -g faces-cli@latest` — older versions have bugs with API key auth and config management that will cause failures.

## 2. Get authenticated

Use AskUserQuestion to figure out where the user is:

> To use Faces, you need an account. Where are you starting from?
>
> A) I have an account — I'll log in now
> B) I have an API key — let me paste it
> C) I need to create an account — help me here
> D) I'll register at faces.sh myself and come back

**If A (log in):**
```bash
faces auth:login --email USER_EMAIL --password 'USER_PASSWORD'
```
Verify: `faces auth:whoami`

**If B (API key):**
```bash
faces config:set api_key <key>
```
Verify: `faces auth:whoami`

**If C (register here):**

Ask which plan via AskUserQuestion:

> Faces has two plans. Which fits you better?
>
> A) **Free** — $5 initial spend (added as API credits), pay per token
>    with a 5% markup. Good for trying it out.
> B) **Connect ($17/month)** — 100k compile tokens/month. If you have
>    ChatGPT Plus or Pro, link it for free gpt-5.x inference.

Then register with the chosen plan:

```bash
# Free plan
RESULT=$(faces auth:register --email USER_EMAIL --password 'USER_PASSWORD' --username USERNAME --json)
echo "$RESULT" | jq -r '.activation_checkout_url'

# Connect plan
RESULT=$(faces auth:register --email USER_EMAIL --password 'USER_PASSWORD' --username USERNAME --plan connect --json)
echo "$RESULT" | jq -r '.activation_checkout_url'
```

Tell the human to open the checkout URL in their browser and complete payment.
Wait for confirmation, then verify:

```bash
faces billing:balance --json | jq '.is_active'
```

If `true`, proceed. If `false`, the payment may not have gone through — ask
them to try the link again.

**If D (register themselves):**

Tell them to register at [faces.sh](https://faces.sh), then come back with
either login credentials or an API key.

## 3. Create a Face

Start by creating the Face with basic demographic attributes. These anchor the persona and improve compilation quality when source material is added later. Common keys: `gender`, `age`, `location`, `occupation`, `education_level`, `religion`, `ethnicity`, `nationality`, `marital_status`. Run `faces face:attributes` for the full list — unsupported keys are dropped with a warning.

```bash
faces face:create --name "Alice Smith" --alias alice \
  --default-model gpt-5-nano \
  --attr gender=female --attr age=29 \
  --attr location="Brooklyn, NY" \
  --attr occupation="product designer" \
  --attr education_level="bachelor's degree" \
  --json
```

Save the face `id` (alias) — you'll need it for the next steps.

## 4. Add source material

Pick one or more methods depending on what the human has:

**Local file (text, PDF):**
```bash
DOC_ID=$(faces compile:upload alice --file /path/to/document.pdf --kind document --no-wait --json | jq -r '.document_id // .id')
```

**YouTube video (solo speaker):**
```bash
IMPORT=$(faces compile:import alice \
  --url "https://www.youtube.com/watch?v=VIDEO_ID" \
  --type document --perspective first-person --no-wait --json)
DOC_ID=$(echo "$IMPORT" | jq -r '.document_id // .doc_id // .id')
```

**YouTube video (multi-speaker interview):**
```bash
IMPORT=$(faces compile:import alice \
  --url "https://www.youtube.com/watch?v=VIDEO_ID" \
  --type thread --no-wait --json)
THREAD_ID=$(echo "$IMPORT" | jq -r '.thread_id // .id')
# After transcription completes, review and remap speaker:
# faces compile:thread:edit "$THREAD_ID" --face-speaker "A"
```

If `--type thread` fails with 422, retry with `--type document`.

**Raw text:**
```bash
DOC_ID=$(faces compile:doc:create alice --label "Notes" --content "Text here..." --json | jq -r '.id')
```

## 5. Compile

For documents (including uploads and YouTube-as-document):

```bash
# Compile an existing document
faces compile:doc:make "$DOC_ID" --no-wait --json
```

Or, if you created the document inline (step 4, "Raw text"), you can create and compile in one step:

```bash
faces compile:doc alice --file essay.txt --no-wait --json
# Poll: faces compile:doc:get DOC_ID --json | jq '{prepare_status}'
```

For threads:

```bash
faces compile:thread:make "$THREAD_ID" --no-wait --json
```

Poll status: `faces compile:thread:get THREAD_ID --json | jq '{prepare_status, chunks_completed, chunks_total}'`

## 6. Chat

`chat:chat` auto-routes to the correct API based on model provider. If a `--default-model` was set on the face, no `--llm` flag is needed.

```bash
# Uses default model (gpt-5-nano set in step 3)
faces chat:chat alice -m "What matters most to you?"

# Override with a specific model
faces chat:chat alice --llm claude-sonnet-4-6 -m "What matters most to you?"
```

## 7. Verify it worked

```bash
faces face:get alice --json | jq '{name, component_counts}'
```

If `component_counts` shows non-null values, the Face is compiled and ready.

## What's next

- **Add more material** — repeat steps 4–5 to deepen the Face. Each compile adds to the existing knowledge.
- **Create more Faces** — repeat steps 3–5 for each persona.
- **Compare Faces** — `faces face:diff --face alice --face bob`
- **Compose Faces** — `faces face:create --alias alice-and-bob --formula "alice | bob"`
- **Use templates** — reference multiple faces in a single prompt: `faces chat:chat gpt-4o-mini -m 'Compare ${alice} and ${bob}.'`
- **Connect ChatGPT** (connect plan) — `faces auth:connect openai` for free gpt-5.x inference. See [OAUTH.md](OAUTH.md).
- **Create an API key** — `faces keys:create --name "my-key"` for programmatic access. See [AUTH.md](AUTH.md).
