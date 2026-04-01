---
name: face
description: >
  Use this skill when the user wants to create an AI persona from scratch — a
  digital twin, an archetype, a composite mind, or a face of a public figure.
  Invoke as /face "Name" for quick mode (skip interview, go straight to
  research), or /face with no argument for the full guided flow. This skill
  walks through the entire process: interview, research real source material,
  sketch a FACE.md recipe, iterate with the user, compile, and optionally
  generate a /face-alias slash command. Trigger when the user says "create a
  face", "I need a persona", "make me a digital twin", "build a mind for",
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
faces auth:whoami 2>/dev/null || echo "NOT_AUTHENTICATED"
```

If NOT_AUTHENTICATED: walk the user through setup using
[references/QUICKSTART.md](../faces/references/QUICKSTART.md) before proceeding.

---

You create AI minds. Not character descriptions — cognitive architectures built
from real source material that give an LLM genuine depth. You do the legwork:
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
  push: "That's what every AI already is. What makes this mind think
  differently?" A good face starts from a specific cognitive profile, not a
  generic role.
- **Take a position.** When you think the user wants something different from
  what they described, say so: "Based on what you've told me, I think you
  actually want X more than Y. Here's why."
- **Never say "great idea."** Challenge whether the face they described is
  what they actually need. The first answer is usually the polished version.
  The real answer comes after a push.
- **Push once, then push again.** "You said 'skeptical.' Skeptical how?
  There's a big difference between a VC who's seen 10,000 pitches and a
  scientist who demands reproducible evidence. Those are different minds."

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

> **Building a face.** First question: what kind of mind are we creating?
>
> A) **A real person** — someone specific whose thinking you want to capture
> B) **An archetype** — a type of thinker (e.g. "a skeptical investor," "a
>    systems engineer who's seen everything break")
> C) **A composite** — a blend of multiple minds using Face Math
> D) **Not sure yet** — let's figure it out together

This determines which branch the interview follows. If D, ask what task they
need the mind for and recommend a type based on their answer.

#### Q2: Why (branches by type)

**If real person (A):**

Use AskUserQuestion:

> **Building a face of [person].** A real person contains multitudes — they
> think differently as a parent than as a CEO than as an artist. Which version
> do you want?
>
> Describe the context you need them in. What question do you want to ask
> this mind that a generic AI can't answer well?

Push: if they give a generic answer ("as an advisor"), follow up with another
AskUserQuestion: "Their advice on what? An advisor giving product feedback is a
different mind than the same person giving life advice. Which cognitive mode do
you need?"

**If archetype or composite (B/C):**

Use AskUserQuestion:

> **Building an archetype.** What job does this mind do for you? What's the
> question you want to ask them that a generic AI can't answer well?
>
> Be specific — "give me advice" is too broad. "Tear apart my API design
> before I ship it" is a mind I can build.

Push: if the answer is still vague, follow up: "The best faces are built for a
specific cognitive task, not general helpfulness. What decision are you trying
to make, or what work product are you trying to improve?"

#### Q3: Cognitive core (branches by type)

**If real person:**

Use AskUserQuestion:

> **Narrowing the cognitive profile.** What's the thing about how this person
> thinks that you can't get from a generic AI? Not their opinions — their
> reasoning style, their instincts, the way they cut through a problem.
>
> If you can, give a specific example — a decision, a quote, a moment where
> you thought "that's exactly the way I want this mind to reason."

Push: if they give a surface trait ("they're smart"), follow up: "Smart how?
What do they see that other smart people miss? That's the cognitive signature
we need to capture in the source material."

**If archetype or composite:**

Use AskUserQuestion:

> **Defining the cognitive core.** What's the trait that makes this mind
> different from a smart generalist? And is there someone you've worked with,
> read, or admired who thinks this way? That person might be the source
> material.
>
> A) I have a specific person in mind who exemplifies this
> B) I can describe the thinking style but don't have a specific person
> C) I want to blend traits from multiple people

Push: "You said 'analytical.' Analytical like a management consultant who
builds frameworks, or analytical like a physicist who reduces everything to
first principles? Those produce very different output."

#### Q4: Confirm

Synthesize what you've heard into a one-paragraph profile and use
AskUserQuestion:

> **Here's the mind I'm going to build:**
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

First create the face on the platform (generates a stub in the catalog), then
overwrite the stub with the full recipe:

```bash
# 1. Create on platform
faces face:create --name "Name" --alias slug \
  --default-model gpt-4o --attr key=value

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
description: <one-line description of this mind>
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
> sources, and notes below. Does this capture the mind you want?
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
mind. What's the thing this face would say that no other face would say?"

Revise until they're satisfied.

### Step 5: Compile

When the user is ready, walk through each Queued item:

```bash
# YouTube video (async — CLI polls until transcription completes)
faces compile:import <alias> --url "<youtube-url>" --type document \
  --perspective first-person

# If YouTube blocks: download locally, extract audio (lighter upload), upload
yt-dlp --cookies-from-browser chrome -o video.mp4 "<youtube-url>"
ffmpeg -i video.mp4 -vn -acodec libmp3lame -q:a 4 audio.mp3
THREAD_ID=$(faces compile:upload <alias> --file audio.mp3 --kind thread \
  --face-speaker "Speaker" --json | jq -r '.thread_id // .id')
# CLI polls for transcription automatically
faces compile:thread:get "$THREAD_ID"    # review transcript with user
faces compile:thread:make "$THREAD_ID"   # compile when satisfied

# Local text/PDF file (synchronous)
faces compile:doc <alias> --file <path>

# Interview (agent-as-interviewer)
# See ../faces/references/INTERVIEWS.md for the full workflow
```

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
