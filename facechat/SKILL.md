---
name: facechat
description: >
  Chat with any face in your catalog. Invoke as /facechat <alias> to start
  chatting directly, or /facechat with no argument to browse your catalog and
  pick a face. Trigger when the user says "chat with a face", "talk to
  <name>", "ask <name>", or wants to have a conversation with a compiled face.
  Also trigger on /facechat.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
  - WebFetch
---

# /facechat — Chat with a Face

## Preamble

```bash
faces --version 2>/dev/null || echo "NOT_INSTALLED"
LATEST=$(npm outdated -g faces-cli --json 2>/dev/null | jq -r '.["faces-cli"].latest // empty')
[ -n "$LATEST" ] && echo "UPDATE_AVAILABLE: $LATEST"
faces auth:whoami --json 2>/dev/null
echo "EXIT:$?"
[ -f ~/.faces/config.json ] && echo "HAS_CONFIG" || echo "NO_CONFIG"
```

If `NOT_INSTALLED`: run `npm install -g faces-cli` and re-run the preamble.

If `UPDATE_AVAILABLE`: run `npm install -g faces-cli@latest` before proceeding.

**Auth triage:**

- `EXIT:0` -> authenticated. Proceed.
- `EXIT:1` + `HAS_CONFIG` -> returning user. Read the whoami output to
  understand what failed. Present the diagnosis and help them fix it. Do NOT
  walk through QUICKSTART or ask about plans.
- `EXIT:1` + `NO_CONFIG` -> new user. Use AskUserQuestion:

  > You're not logged into Faces, and I don't see any prior config on this
  > machine. Do you already have an account?
  >
  > A) I have an account -- I'll log in now
  > B) I have an API key -- let me paste it
  > C) No account -- help me set one up here
  > D) No account -- I'll register at faces.sh myself and come back

  If A: prompt `! faces auth:login --email YOUR_EMAIL --password 'YOUR_PASSWORD'`
  If B: prompt `! faces config:set api_key <key>`, verify with `faces auth:whoami`
  If C: walk through [QUICKSTART.md](../faces/references/QUICKSTART.md)
  If D: tell them to come back with login credentials or an API key

**Secret hygiene:** Never display API keys, tokens, or passwords from config
files. Always mask them (e.g. `sk-faces-...dN`).

---

## Resolve the face

**If alias provided** (`/facechat albert`):

Check that the face exists:

```bash
faces face:get $ALIAS --json 2>/dev/null
```

If not found, check for close matches:

```bash
cat ~/.faces/catalog.json | jq -r '.[].alias'
```

Suggest the closest match or offer to create the face with `/face`.

**If no alias provided** (`/facechat`):

List the catalog and let the user pick:

```bash
faces face:list --json 2>/dev/null
```

Present faces as a numbered list showing alias, name, and description. Use
AskUserQuestion:

> **Your face catalog.** Who do you want to chat with?
>
> [numbered list of faces]
>
> Pick a number, or type a name/alias.

## Chat

Once the face is resolved, relay the user's message:

```bash
faces chat:chat ALIAS -m "MESSAGE"
```

If the user didn't include a message with the alias, ask what they want to
discuss. Use AskUserQuestion:

> **Chatting with NAME.** What would you like to ask or discuss?

### Conversation loop

After each response from the face, the user can:
- Send another message (continue the conversation)
- Type `/facechat` again to switch to a different face
- Move on to other work

For follow-up messages in the same conversation, keep using
`faces chat:chat ALIAS -m "MESSAGE"` with the same alias.

### Model override

If the user wants a different model than the face's default, pass `--llm`:

```bash
faces chat:chat ALIAS -m "MESSAGE" --llm MODEL
```

### Template references

Users can reference other faces inline with `${other-alias}` syntax. Pass the
message as-is -- the platform handles template expansion.

## Related skills

- `/face` -- Guided face creation
- `/faces` -- Core CLI for all platform operations
- `/faceteam` -- Compose faces into teams
