---
name: faces
description: >
  Use this skill whenever someone wants to build an AI that thinks like, sounds
  like, or acts like a specific person. This includes: turning interviews,
  letters, videos, or documents into a chateable AI persona; creating customer
  personas or focus groups you can actually talk to (not just static profiles);
  replacing bloated SOUL.md / soul documents / system prompts with something
  more token-efficient; giving agents in a multi-agent system distinct
  personalities, behavioral modes, or points of view; creating a digital twin
  or memorial AI; comparing how different people think; composing new
  perspectives from existing ones; or sharing persona access via scoped API
  keys. Also trigger when the user mentions the Faces Platform, the `faces`
  CLI, cognitive primitives, mind arithmetic, or compile personality. Use even
  if Faces is not yet installed — the skill covers setup. Do NOT use for
  fine-tuning models, RAG/retrieval systems, creative writing (NPC backstories,
  fiction), or thematic analysis of transcripts.
compatibility: Requires the faces CLI (npm install -g faces-cli) and internet access to api.faces.sh.
---

# Faces Skill

The Faces Platform compiles source texts (documents and conversation transcripts) into a Face — a persona built of cognitive primitives that give an underlying LLM a complex perspective that is richer, more accurate, and uses far fewer tokens than prompt-stuffing approaches. Faces can be chatted with, compared, and composed.

Official docs: https://docs.faces.sh

Always use `--json` when you need to extract values from command output.

## Current config
!`faces config:show 2>/dev/null || echo "(no config saved)"`

## Setup

Check CLI: `faces --version`. Install if missing: `npm install -g faces-cli`.
If below v1.4.4: `npm install -g faces-cli@latest` (older versions have auth bugs).

Check credentials: `faces auth:whoami`. If none exist, see
[references/QUICKSTART.md](references/QUICKSTART.md) for full setup.

## Core workflows

### 1. Create a Face

```bash
faces face:create --name "Name" --alias slug --default-model gpt-5-nano \
  --attr gender=male --attr age=34 --attr location="Portland, OR" \
  --attr occupation="nurse practitioner"
```

Common `--attr` keys: gender, age, location, occupation, education_level,
religion, ethnicity, nationality, marital_status.
Full list → [references/ATTRIBUTES.md](references/ATTRIBUTES.md) (unrecognized keys silently ignored).

### 2. Add source material

**Interview** (no documents needed — best way to build a face from scratch):
Run a Q&A interview with the user, save the transcript, upload and compile.
See [references/INTERVIEWS.md](references/INTERVIEWS.md) for both modes (agent-as-interviewer recommended, built-in interviewer also available).

**Document (essay, notes, PDF):**
```bash
faces compile:doc alias --file document.txt
```

**YouTube solo talk → document:**
```bash
faces compile:import alias --url "URL" --type document --perspective first-person
```

**YouTube multi-speaker → thread:**
```bash
faces compile:import alias --url "URL" --type thread --face-speaker A
```

**Upload a local file (text, PDF, audio, video):**
```bash
# Document (text/PDF — synchronous)
DOC_ID=$(faces compile:upload alias --file report.pdf --kind document --json | jq -r '.document_id // .id')
faces compile:doc:make "$DOC_ID"

# Thread from text transcript
THREAD_ID=$(faces compile:upload alias --file transcript.txt --kind thread --face-speaker "Name" --json | jq -r '.thread_id // .id')
faces compile:thread:make "$THREAD_ID"

# Thread from audio/video — async, CLI polls automatically
THREAD_ID=$(faces compile:upload alias --file recording.mp4 --kind thread --face-speaker "Guest" --json | jq -r '.thread_id // .id')
# CLI polls until transcription completes, then returns
# Review the transcript before compiling:
faces compile:thread:get "$THREAD_ID"
# When satisfied:
faces compile:thread:make "$THREAD_ID"
```

Audio/video uploads and imports are asynchronous — the server returns 202
immediately and transcribes in the background. The CLI polls automatically
until done. Always review the transcript before compiling.

Status lifecycle for audio/video:
`transcribing` → `null` (ready) → `preparing` → `syncing` → `synced`

**If YouTube blocks the download** ("Sign in to confirm you're not a bot"):
`compile:import` downloads server-side and can't use your browser cookies.
Download the video locally with yt-dlp and upload it instead:
```bash
yt-dlp --cookies-from-browser chrome -o episode.mp4 "https://youtube.com/watch?v=VIDEO_ID"
THREAD_ID=$(faces compile:upload alias --file episode.mp4 --kind thread --face-speaker "Guest" --json | jq -r '.thread_id // .id')
# CLI polls for transcription automatically
faces compile:thread:get "$THREAD_ID"    # review transcript
faces compile:thread:make "$THREAD_ID"   # compile when ready
```
Use `--kind document` for solo speakers. For diarized audio, speakers are labeled A, B, etc.

### 3. Chat

```bash
faces chat:chat alias -m "message"
```

Auto-routes by model provider. Override: `--llm MODEL`.
Reference other faces inline: `${other-alias}` → [references/TEMPLATES.md](references/TEMPLATES.md).

### 4. Compare & compose

```bash
faces face:diff --face alice --face bob
faces face:neighbors alias --k 3
faces face:create --alias new --formula "alice | bob"
```

Operators: `|` union, `&` intersection, `-` difference, `^` symmetric diff.

## Important: never run `config:clear`

`faces config:clear` wipes all stored credentials (API key, JWT, base URL) with no recovery. Never use it to troubleshoot auth issues. If auth fails, check `faces config:show` and fix the specific value with `faces config:set`.

## Common errors

| Error | Fix |
|---|---|
| `faces: command not found` | `npm install -g faces-cli` |
| `401 Unauthorized` | `faces auth:login` or check `FACES_API_KEY` via `faces config:show` |
| status "transcribing" | Audio/video transcription in progress — CLI polls automatically, just wait |
| status "preparing" | Compilation in progress — poll with `compile:doc:get ID --json` or `compile:thread:get ID --json` |
| `422` on thread import | Retry with `--type document` |

## Related skills

- `/face` — Guided face creation (interview, research, sketch, compile, generate slash command)
- `/faceteam` — Compose faces into teams with collaboration protocols
- `/manyface` — Transform a skill into a multi-persona version

## Filesystem

```
~/.faces/
  catalog.json          # auto-generated index
  catalog/<alias>/      # individual FACE.md files
  teams/<team-name>/    # TEAM.md files (collaboration protocols)
  skills/face-<alias>/  # auto-generated face slash commands
```

## References

- [QUICKSTART.md](references/QUICKSTART.md) — First-time setup end-to-end
- [REFERENCE.md](references/REFERENCE.md) — Full CLI command reference
- [AUTH.md](references/AUTH.md) — Registration, login, API keys, plans
- [INTERVIEWS.md](references/INTERVIEWS.md) — Interview & conversation workflows
- [CONCEPTS.md](references/CONCEPTS.md) — What Faces is, cognitive primitives
- [USE_CASES.md](references/USE_CASES.md) — 14 example applications
- [TEMPLATES.md](references/TEMPLATES.md) — Face template syntax
- [ATTRIBUTES.md](references/ATTRIBUTES.md) — Accepted --attr keys
- [OAUTH.md](references/OAUTH.md) — ChatGPT account linking
- [SCOPE.md](references/SCOPE.md) — Security boundaries
