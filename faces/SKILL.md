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

Without a Face, an LLM is **faceless** — generic, interchangeable, forgettable. A Face specializes it. Gives it perspective, coherence, and nuance that doesn't collapse back into LLM-speak on long threads.

Official docs: https://docs.faces.sh

Always use `--json` when you need to extract values from command output.

## Current config
!`faces config:show 2>/dev/null || echo "(no config saved)"`

## Setup

Check CLI: `faces --version`. Install if missing: `npm install -g faces-cli`.
If below v1.4.4: `npm install -g faces-cli@latest` (older versions have auth bugs).

Check credentials: run `faces config:show` to see active credentials.
- API key takes priority over JWT — if both exist, the API key is used
- If switching environments (localhost vs production, different accounts),
  clear the stale credential: `faces config:set api_key ""` or `faces config:set token ""`
- When in doubt, pass `--api-key` and `--base-url` explicitly
- Never run `faces config:clear` (wipes everything with no recovery)

If no credentials exist, see [references/QUICKSTART.md](references/QUICKSTART.md) for full setup.

## Core workflows

### 1. Create a Face

```bash
faces face:create --name "Name" --alias slug --default-model MODEL \
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
faces compile:doc alias --file document.txt --no-wait --json
# Poll: faces compile:doc:get DOC_ID --json | jq '{prepare_status}'
```

**YouTube solo talk → document:**
```bash
faces compile:import alias --url "URL" --type document --perspective first-person --no-wait --json
# Returns doc_id immediately. Poll for transcription then compilation.
```

**YouTube multi-speaker → thread:**
```bash
faces compile:import alias --url "URL" --type thread --no-wait --json
# Returns thread_id. After transcription completes, remap speaker:
faces compile:thread:edit THREAD_ID --face-speaker "A"
faces compile:thread:make THREAD_ID --no-wait --json
```

**Upload a local file (text, PDF, audio, video):**
```bash
# Document (text/PDF)
DOC_ID=$(faces compile:upload alias --file report.pdf --kind document --no-wait --json | jq -r '.document_id // .id')
faces compile:doc:make "$DOC_ID" --no-wait --json

# Thread from text transcript (you know the speakers — pass --face-speaker)
THREAD_ID=$(faces compile:upload alias --file transcript.txt --kind thread --face-speaker "Name" --no-wait --json | jq -r '.thread_id // .id')
faces compile:thread:make "$THREAD_ID" --no-wait --json

# Thread from audio/video — DON'T pass --face-speaker at upload
THREAD_ID=$(faces compile:upload alias --file recording.mp4 --kind thread --no-wait --json | jq -r '.thread_id // .id')
# Poll for transcription:
faces compile:thread:get "$THREAD_ID" --json | jq '{prepare_status}'
# When transcription done (prepare_status: null), review and remap:
faces compile:thread:get "$THREAD_ID"
faces compile:thread:edit "$THREAD_ID" --face-speaker "B"
faces compile:thread:make "$THREAD_ID" --no-wait --json
```

**Always use `--no-wait`** for compile and upload operations. Each operation
runs independently on the server — you can fire multiple compiles in parallel
without waiting for any to finish. Upload all sources, kick off all compiles,
then poll them all at the end. Poll on your own schedule:
```bash
faces compile:thread:get ID --json | jq '{prepare_status, chunks_completed, chunks_total}'
faces compile:doc:get ID --json | jq '{prepare_status}'
```

Status meanings (`prepare_status` field):

| Status | Meaning | Action |
|--------|---------|--------|
| `"transcribing"` | Audio/video being transcribed | Keep polling |
| `null` | Ready to compile (transcription done, no compilation started) | Run `compile:thread:make` or `compile:doc:make` |
| `"preparing"` | Compilation in progress, extracting cognitive primitives | Keep polling |
| `"syncing"` | Writing primitives to the face | Almost done, keep polling |
| `"synced"` | Done | Compilation complete |
| `"paused"` | Compilation paused by user/agent | Resume with `compile:*:make` or reset with `compile:*:reset` |
| `"failed"` | Something went wrong | Investigate or retry |
| `"stalled"` | Stuck for 10+ minutes | Re-run the make command |

**If YouTube blocks the download** ("Sign in to confirm you're not a bot"):
Download locally with yt-dlp, extract and compress audio with ffmpeg, upload.
Keep files under 100MB — large uploads can fail. For long recordings (1hr+),
use mono 48kbps:
```bash
yt-dlp --cookies-from-browser chrome -o episode.mp4 "https://youtube.com/watch?v=VIDEO_ID"
ffmpeg -i episode.mp4 -vn -ac 1 -b:a 48k episode.mp3
THREAD_ID=$(faces compile:upload alias --file episode.mp3 --kind thread --no-wait --json | jq -r '.thread_id // .id')
# Poll for transcription, then review, remap, compile:
faces compile:thread:get "$THREAD_ID"
faces compile:thread:edit "$THREAD_ID" --face-speaker "A"
faces compile:thread:make "$THREAD_ID" --no-wait --json
```
Use `--kind document` for solo speakers.

**`--face-speaker` label rules:**
- Audio/video transcription: speakers are labeled `A`, `B`, `C` — use the short label (e.g. `--face-speaker B`)
- Text transcript uploads: the label matches the speaker name in the file (e.g. `--face-speaker Troy`)
- Match is case-insensitive but exact

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
| status "transcribing" | Audio/video transcription in progress — poll with `compile:thread:get ID --json` |
| status "preparing" | Compilation in progress — poll with `compile:doc:get ID --json` or `compile:thread:get ID --json` |
| `422` on thread import | Retry with `--type document` |
| Bad extraction results | Pause with `compile:thread:pause ID` or `compile:doc:pause ID`, review what was extracted, then either resume with `compile:*:make ID` or wipe and restart with `compile:*:reset ID` (keeps source content, removes extraction) |

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
- [CONTRIBUTING.md](references/CONTRIBUTING.md) — File bug reports via `gh issue create`
