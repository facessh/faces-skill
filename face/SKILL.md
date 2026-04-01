---
name: face
description: >
  Use this skill when the user wants to create an AI persona from scratch — a
  digital twin, an archetype, a composite mind, or a face of a public figure.
  Invoke as /face "Garry Tan" for quick mode (skip interview, go straight to
  research), or /face with no argument for the full guided flow. This skill
  walks through the entire process: interview, research real source material,
  sketch a FACE.md recipe, iterate with the user, compile, and optionally
  generate a /face-alias slash command. Trigger when the user says "create a
  face", "I need a persona", "make me a digital twin", "build a mind for",
  or names a person they want to turn into an AI. Also trigger on /face.
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

## The flow

### Step 1: Interview

**Quick mode** — user provides a name or role inline:
`/face "Garry Tan"` or `/face "skeptical Series A investor"`

Skip the interview. Determine if this is a public figure (search for them) or
an archetype (identify exemplars). Go straight to Step 2.

**Full mode** — no argument. Ask:
- Who is this face? (a real person, an archetype, a composite?)
- What task or role do they fill?
- What cognitive traits matter most?
- Any specific people who exemplify what you're looking for?

Keep it to 3-5 questions. You're casting, not therapizing.

### Step 2: Research

Before creating anything, check the catalog:

```bash
cat ~/.faces/catalog.json
```

If a suitable face exists, recommend reuse over duplication. If you need more
detail on a candidate: `cat ~/.faces/catalog/<alias>/FACE.md`

For new faces, use WebSearch to find real source material:

**Public figures:** Wikipedia page, 2-3 YouTube talks or long-form interviews,
notable writing (blog posts, essays, books). Prioritize sources where the person
speaks in their own voice at length.

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

- [ ] <url> — <what this source contributes>

## Sources

## Lessons

## Notes

<why this face was cast, cognitive traits that matter, etc.>
```

### Step 4: Iterate

Present the FACE.md to the user. This is co-creation — they may want to:
- Add or remove sources from Queued
- Change the description or role
- Adjust the cognitive traits in Notes
- Switch from archetype to public-figure or vice versa

Revise until they're satisfied. The FACE.md is a design doc, not a one-shot
output.

### Step 5: Compile

When the user is ready, walk through each Queued item:

```bash
# YouTube video
faces compile:import <alias> --url "<youtube-url>" --type document \
  --perspective first-person

# Local file
faces compile:doc <alias> --file <path>

# Interview (agent-as-interviewer)
# See ../faces/references/INTERVIEWS.md for the full workflow
```

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
