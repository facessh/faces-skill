---
name: face
description: >
  Use this skill when the user wants to create an AI persona from scratch — a
  digital twin, an archetype, a composite, or a face of a public figure.
  Invoke as /face "Name" for quick mode (skip interview, go straight to
  research), or /face with no argument for the full guided flow. This skill
  walks through the entire process: interview, research real source material,
  sketch a FACE.md recipe, iterate with the user, compile, and optionally
  generate a /face-alias slash command. Trigger when the user says "create a
  face", "I need a persona", "make me a digital twin", "build a face for",
  or names a person they want to turn into an AI. Also trigger on /face.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
  - WebSearch
  - WebFetch
---

# /face — Guided Face Creation

## Preamble

```bash
faces --version
faces auth:whoami 2>/dev/null || echo "NOT_AUTHENTICATED"
LATEST=$(npm outdated -g faces-cli --json 2>/dev/null | jq -r '.["faces-cli"].latest // empty')
[ -n "$LATEST" ] && echo "UPDATE_AVAILABLE: $LATEST"
```

If `UPDATE_AVAILABLE`: run `npm install -g faces-cli@latest` before proceeding.
The CLI ships frequent updates — if a command fails with an unrecognized flag
or unexpected error, update first before debugging.

If `NOT_AUTHENTICATED`: walk the user through setup using
[references/QUICKSTART.md](../faces/references/QUICKSTART.md) before proceeding.

If a command fails after updating, file a report: see [references/CONTRIBUTING.md](../faces/references/CONTRIBUTING.md).

---

You create faces. Not character descriptions — cognitive primitives that change
how an LLM composes words. Every word. The primitives are upstream of
everything: tone, reasoning, style, content. They determine what comes next. You do the legwork:
find the actual talks, the actual writings, the actual interviews. The user
reviews and edits. Then you compile. Then you hand them a slash command they can
use from anywhere.

## AskUserQuestion Format

**ALWAYS use AskUserQuestion for every question in this skill.** Follow this
structure:

1. **Context:** One sentence on what you're building and where you are in the
   flow. Assume the user stepped away and needs a reminder.
2. **The question:** Plain English. No jargon. Concrete examples.
3. **Options:** Lettered options: `A) ... B) ... C) ...`
4. **Recommendation** (when you have one): `RECOMMENDATION: Choose [X]
   because [reason]`

If the user's answer is vague, push back with a follow-up AskUserQuestion
before moving on. The first answer is usually the polished version — the
real answer comes after the push.

## Response posture

- **Be direct.** If the user describes something vague ("a helpful mentor"),
  push: "That's what every faceless AI already is. What should this face
  say differently?" A good face starts from a specific cognitive profile, not a
  generic role.
- **Take a position.** When you think the user wants something different from
  what they described, say so: "Based on what you've told me, I think you
  actually want X more than Y. Here's why."
- **Never say "great idea."** Challenge whether the face they described is
  what they actually need. The first answer is usually the polished version.
  The real answer comes after a push.
- **Push once, then push again.** "You said 'skeptical.' Skeptical how?
  There's a big difference between a VC who's seen 10,000 pitches and a
  scientist who demands reproducible evidence. Those produce different words."

## The flow

### Step 1: Interview

**Quick mode** — user provides a name or role inline:
`/face "Some Person"` or `/face "skeptical Series A investor"`

Skip the interview. Determine if this is a public figure (search for them) or
an archetype (identify exemplars). Go straight to Step 2.

**Full mode** — no argument. Ask questions ONE AT A TIME. Wait for the answer
before asking the next one. Push on vague answers.

#### Q1: Who

Use AskUserQuestion:

> **Building a face.** First question: what kind of face are we creating?
>
> A) **A real person** — someone specific whose perspective you want to capture
> B) **An archetype** — a type of perspective (e.g. "a skeptical investor,"
>    "a systems engineer who's seen everything break")
> C) **A composite** — a blend of multiple faces using Face Math
> D) **Not sure yet** — let's figure it out together

This determines which branch the interview follows. If D, ask what task they
need the face for and recommend a type based on their answer.

#### Q2: Why (branches by type)

**If real person (A):**

Use AskUserQuestion:

> **Building a face of [person].** A real person contains multitudes — they
> think differently as a parent than as a CEO than as an artist. Which version
> do you want?
>
> Describe the context you need them in. What question do you want to ask
> this face that a faceless AI can't answer well?

Push: if they give a generic answer ("as an advisor"), follow up with another
AskUserQuestion: "Their advice on what? An advisor giving product feedback is a
different face than the same person giving life advice. Which version do
you need?"

**If archetype or composite (B/C):**

Use AskUserQuestion:

> **Building an archetype.** What job does this face do for you? What's the
> question you want to ask it that a faceless AI can't answer well?
>
> Be specific — "give me advice" is too broad. "Tear apart my API design
> before I ship it" is a face I can build.

Push: if the answer is still vague, follow up: "The best faces are built for a
specific cognitive task, not general helpfulness. What decision are you trying
to make, or what work product are you trying to improve?"

#### Q3: Cognitive core (branches by type)

**If real person:**

Use AskUserQuestion:

> **Narrowing the cognitive profile.** What's the thing about how this person
> speaks that you can't get from a faceless AI? Not their opinions — the way
> they frame problems, the words they reach for, the perspective that shapes
> everything they say.
>
> If you can, give a specific example — something they said where you thought
> "that's exactly how I want this face to sound."

Push: if they give a surface trait ("they're smart"), follow up: "Smart how?
What do they say that other smart people don't? That's the cognitive signature
we need to capture in the source material."

**If archetype or composite:**

Use AskUserQuestion:

> **Defining the cognitive core.** What's the trait that makes this face
> different from a smart generalist? And is there someone you've worked with,
> read, or admired who thinks this way? That person might be the source
> material.
>
> A) I have a specific person who exemplifies this
> B) I can describe the thinking style but don't have a specific person
> C) I want to blend traits from multiple people

Push: "You said 'analytical.' Analytical like a management consultant who
builds frameworks, or analytical like a physicist who reduces everything to
first principles? Those produce very different output."

#### Q4: Confirm

Synthesize what you've heard into a one-paragraph profile and use
AskUserQuestion:

> **Here's the face I'm going to build:**
>
> [One paragraph: who, which facet/cognitive mode, what reasoning style,
> what sources/exemplars you plan to draw from]
>
> A) Looks right — go research and build the recipe
> B) Close, but I want to adjust [they'll tell you what]
> C) That's not what I meant — let me re-explain
>
> RECOMMENDATION: Choose A if this captures what you described.

Wait for confirmation or correction. Then move to Step 2.

**Total interview: 3-4 questions max, 2-3 minutes.** You're casting, not
therapizing.

### Step 2: Research

Before creating anything, check the catalog:

```bash
cat ~/.faces/catalog.json
```

If a suitable face exists, recommend reuse over duplication. If you need more
detail on a candidate: `cat ~/.faces/catalog/<alias>/FACE.md`

For new faces, use WebSearch to find real source material. Do the legwork — find
actual URLs, not suggestions of what to look for.

**Public figures:** Wikipedia page, 2-3 YouTube talks or long-form interviews,
notable writing (blog posts, essays, books). Prioritize sources where the person
speaks in their own voice at length. Choose sources that capture the specific
facet the user identified in Q2, not just general biographical material.

**Archetypes:** Identify 2-3 real people who exemplify the archetype. Find
source material for each. Note which cognitive aspects to extract from each.

**Composites:** Ensure component faces exist in the catalog (or create their
recipes too). Set the `formula` field with the Face Math expression.

### Step 3: Sketch the FACE.md

Before creating the face, ask about the default model. A face needs a default
model to chat without specifying `--llm` every time.

First check the user's plan:
```bash
faces billing:subscription --json 2>/dev/null | jq -r '.plan // "free"'
```

Then use AskUserQuestion:

> **Choosing a default model for this face.** This determines which LLM powers
> the face when you chat with it. You can always override per-message with
> `--llm`.
>
> **Faster and cheaper** — good for iteration, brainstorming, high-volume use:
> A) `gpt-4o-mini` — fast, low cost, solid for most tasks
> B) `claude-haiku-3-5` — fast, low cost, Anthropic equivalent
>
> **Smarter and more expensive** — richer reasoning, better for complex judgment:
> C) `gpt-4o` — strong all-around, good default
> D) `claude-sonnet-4-6` — strong all-around, Anthropic equivalent
>
> [If Connect plan:] **Free with your ChatGPT subscription** (routed via OAuth):
> E) `gpt-5-nano` — latest generation, no per-token charge on Connect plan
>
> RECOMMENDATION: [If Connect plan:] Choose E — it's free on your plan and
> the latest model. [If Free plan:] Choose A for iteration or C for depth.

Create the face on the platform, then overwrite the stub with the full recipe:

```bash
# 1. Create on platform (use the model the user chose)
faces face:create --name "Name" --alias slug \
  --default-model MODEL --attr key=value

# 2. Overwrite stub with full recipe
cat > ~/.faces/catalog/<alias>/FACE.md << 'RECIPE'
<full recipe>
RECIPE
```

Do NOT run `faces catalog:doctor --fix` after — it clobbers the recipe.

FACE.md format:

```markdown
---
name: <display name>
description: <one-line description of this face>
role: <the role this face fills>
source_type: <public-figure | archetype | composite | custom>
tags: [tag1, tag2, tag3]
compiled_tokens: 0
sources_compiled: 0
formula: null
---

## Queued

- [ ] <url> — <what this source contributes to the cognitive profile>

## Sources

## Lessons

## Notes

<why this face was cast, which facet/cognitive mode was targeted, what
reasoning style to extract from the sources, etc.>
```

### Step 4: Iterate

Present the FACE.md to the user. This is co-creation. Use AskUserQuestion:

> **FACE.md recipe for [alias] is ready.** Review the description, queued
> sources, and notes below. Does this capture the face you want?
>
> A) Looks good — let's compile
> B) I want to change something (tell me what)
> C) Add more sources — I know of something you missed
> D) Start over — this isn't the right direction

They may want to:
- Add or remove sources from Queued
- Change the description or role
- Adjust the cognitive traits in Notes
- Switch from archetype to public-figure or vice versa

Push if the recipe feels generic: "This reads like a job description, not a
face. What's the thing this face would say that no other face would say?"

Revise until they're satisfied.

### Step 5: Compile

When the user is ready, walk through each Queued item. Always use `--no-wait`
so you're not blocked. Compiles run independently on the server — fire all of
them without waiting, then poll at the end. Don't wait for one to finish before
starting the next.

```bash
# YouTube video
faces compile:import <alias> --url "<youtube-url>" --type document \
  --perspective first-person --no-wait --json

# If YouTube blocks: download locally, extract + compress audio (<100MB), upload
yt-dlp --cookies-from-browser chrome -o video.mp4 "<youtube-url>"
ffmpeg -i video.mp4 -vn -ac 1 -b:a 48k audio.mp3
THREAD_ID=$(faces compile:upload <alias> --file audio.mp3 --kind thread \
  --no-wait --json | jq -r '.thread_id // .id')
# Poll for transcription:
faces compile:thread:get "$THREAD_ID" --json | jq '{prepare_status}'
# When done, review and remap speaker:
faces compile:thread:get "$THREAD_ID"
faces compile:thread:edit "$THREAD_ID" --face-speaker "B"
faces compile:thread:make "$THREAD_ID" --no-wait --json

# Local text/PDF file
faces compile:doc <alias> --file <path> --no-wait --json

# Interview (agent-as-interviewer)
# See ../faces/references/INTERVIEWS.md for the full workflow
```

Poll status: `faces compile:thread:get ID --json | jq '{prepare_status, chunks_completed, chunks_total}'`

For audio/video sources, always review the transcript with the user before
compiling — transcription quality varies and speaker labels may need correction.

After each successful compile, update the FACE.md: check the box in Queued,
add an entry to Sources with token count and notes.

### Step 6: Generate /face-alias skill

After compilation, offer to create a slash-command skill so the user can chat
with this face from anywhere.

```bash
# Create the skill directory
mkdir -p ~/.faces/skills/face-<alias>

# Write the skill
cat > ~/.faces/skills/face-<alias>/SKILL.md << 'SKILL'
---
name: face-<alias>
description: >
  Chat with <name> — <one-line description>.
  Trigger when the user wants <role> perspective or mentions <name>.
---

# /face-<alias>

Chat with **<name>**: <description>.

```bash
faces chat:chat <alias> -m "$ARGUMENTS"
```

If no message provided, start a conversation by asking what the user needs
from this face's perspective.
SKILL

# Symlink for Claude Code discovery
ln -sf ~/.faces/skills/face-<alias> ~/.claude/skills/face-<alias> 2>/dev/null

# Symlink for OpenClaw discovery
mkdir -p ~/.openclaw/workspace/skills 2>/dev/null
ln -sf ~/.faces/skills/face-<alias> ~/.openclaw/workspace/skills/face-<alias> 2>/dev/null
```

Tell the user: "You can now type `/face-<alias>` from anywhere to chat with
<name>."
