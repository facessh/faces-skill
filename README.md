# faces-skill

> Every AI agent sounds the same. Helpful. Direct. Forgettable.

The Faces Platform compiles source texts — documents, interviews, YouTube talks — into cognitive primitives that give an LLM a genuine perspective. Not a character description. Not a system prompt. A compressed cognitive architecture that is richer, more accurate, and uses far fewer tokens than prompt-stuffing approaches like SOUL.md files.

**faces-skill** is a bundle of four slash commands that turn Claude Code (or OpenClaw) into a face creation and orchestration system:

- **`/face`** — Create a face from scratch. Interview, research, compile, chat.
- **`/faceteam`** — Compose multiple faces into a team with a collaboration protocol.
- **`/manyface`** — Transform any agent skill into a multi-persona version where every step is run by the ideal face for the job.
- **`/faces`** — Core CLI operations. The power-user interface for direct platform access.

**Who this is for:**
- **Anyone tired of flat AI personas** — Character.ai characters that lose their voice after three messages. Custom GPTs that sound like every other Custom GPT. SOUL.md files that balloon to 2,000 tokens and still produce generic output. Faces compiles cognition, not descriptions.
- **People frustrated with agent orchestration complexity** — If you've tried n8n, Paperclip, CrewAI, or similar tools and found them overengineered for what should be simple ("I just want three different perspectives on this decision"), Faces gives you that with slash commands, not flowchart builders.
- **Isolated thinkers who need a real sparring partner** — Researchers, founders, writers, autodidacts working alone who need something that can genuinely push back on their thinking — not agree politely, not "have you considered the other side," but actually reason from a different cognitive structure.
- **Agent skill authors** — who want to give their skills cognitive depth instead of a generic voice
- **Teams** — who want to build a catalog of reusable faces that compound across projects

## Quickstart (Agent Install)

Paste the following into the chat with your agent:

Use this for **Claude Code**:

```
Install Faces skill:
1. git clone --single-branch --depth 1 https://github.com/facessh/faces-skill.git ~/.claude/skills/faces-skill && cd ~/.claude/skills/faces-skill && ./setup
2. list the available skills: /faces, /face, /faceteam, /manyface
3. set me up to use faces: ask if I have an account, else help me get one
```

Use this for **OpenClaw**:

```
Please install Faces:
1. clawhub install faces
2. run QUICKSTART.md exactly
3. tell me how to use it
```

## Manual Install

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Node.js](https://nodejs.org/) (for the faces CLI)

### Step 1: Install the faces CLI

```bash
npm install -g faces-cli
```

### Step 2: Register an account

You can register manually at the [faces.sh](https://faces.sh/) website, or via the cli:

```bash
# Free plan ($5 activation, pay-per-token)
faces auth:register --email you@example.com --password 'your-password' --username your-username

# Connect plan ($17/month, 100k compile tokens, ChatGPT passthrough)
faces auth:register --email you@example.com --password 'your-password' --username your-username --plan connect
```

This returns a Stripe Checkout URL. Complete payment in your browser, then verify:

```bash
faces billing:balance --json | jq '.is_active'
```

### Step 3: Install the skill bundle

```bash
git clone --depth 1 https://github.com/facessh/faces-skill.git ~/.claude/skills/faces-skill
cd ~/.claude/skills/faces-skill && ./setup
```

The setup script symlinks each skill into `~/.claude/skills/` so Claude Code discovers them. You now have `/face`, `/faceteam`, `/manyface`, and `/faces` available as slash commands.

### OpenClaw

```bash
git clone --depth 1 https://github.com/facessh/faces-skill.git ~/.openclaw/workspace/skills/faces-skill
cd ~/.openclaw/workspace/skills/faces-skill && ./setup
```

## See it work

### Create a face

```
You:    /face "Garry Tan"
Claude: [searches for Garry Tan — finds Wikipedia, YouTube talks, blog posts]
        [checks catalog — no existing match]
        [creates face on platform, writes FACE.md recipe with 6 queued sources]

        Recipe ready at ~/.faces/catalog/garry-tan/FACE.md
        Sources queued:
        - [ ] Wikipedia bio — career trajectory, YC timeline
        - [ ] "Unconventional Advice for Founders" talk — core philosophy
        - [ ] Lightcone Ep. 42 — AI agent thesis
        - [ ] blog.garrytan.com — written voice, long-form thinking
        Review and edit, then I'll compile.

You:    Looks good, compile it.
Claude: [compiles Wikipedia bio — 12.4k tokens, 34 primitives]
        [imports YouTube talk — 18.2k tokens, 48 primitives]
        Done. 82 cognitive primitives across 2 sources.
        Want me to create /face-garry-tan so you can chat from anywhere?

You:    Yes.
Claude: Created /face-garry-tan. Try it now.

You:    /face-garry-tan "Should I pivot from horizontal to vertical SaaS?"
Garry:  Before I answer that — what's your retention look like on the
        horizontal product? Because the thing people get wrong about
        pivots is they think it's about the market. It's not. It's
        about whether you've actually learned something from the
        customers you already have...
```

### Build a team

```
You:    /faceteam "I need an advisory board to evaluate my pivot decision"
Claude: [checks catalog — finds garry-tan, creates 2 new faces]
        [designs round-robin protocol with mermaid diagram]

        Team: pivot-advisory-board
        Protocol: round-robin (3 rounds max)

        ┌─────────────┐    ┌──────────────┐    ┌─────────────┐
        │  garry-tan   │ →  │ domain-expert │ →  │   skeptic   │ → consensus?
        │ product-fit  │    │  feasibility  │    │ assumptions │
        └─────────────┘    └──────────────┘    └─────────────┘

        TEAM.md saved to ~/.faces/faceteams/pivot-advisory-board/
```

### Manyface a skill

```
You:    /manyface
Claude: Do you have a skill or start from scratch?

You:    I have a code review skill. [provides path]
Claude: [reads the skill, decomposes into 5 steps]

        Decomposition:
        Step 1: Read diff           → faceless (mechanical)
        Step 2: Security review     → solo: jim-manico (OWASP mindset)
        Step 3: Code quality        → team pipeline: fowler → uncle-bob
        Step 4: Architecture review → (part of pipeline above)
        Step 5: Write summary       → solo: review-synthesizer

        Reused from catalog: retro-code-quality (Martin Fowler)
        New faces: jim-manico, uncle-bob, review-synthesizer

        manyfaced-review/SKILL.md written.
        3 faces need compilation before use.
```

## The skills

| Skill | What it does |
|-------|-------------|
| `/face` | **Guided face creation.** Six steps: interview → research (real URLs via web search) → sketch FACE.md → iterate with user → compile → generate `/face-alias` slash command. Quick mode: `/face "Person Name"` skips the interview. |
| `/faceteam` | **Team composition.** Composes faces into a team with a collaboration protocol defined by a mermaid flowchart. Five protocol types: round-robin, pipeline, chief-of-staff, debate, voting. Produces a TEAM.md in `~/.faces/faceteams/`. |
| `/manyface` | **Skill orchestrator.** Takes an existing skill (or designs one from scratch), decomposes it into steps, determines which need a solo face vs. a team vs. no face, and outputs a `manyfaced-` skill directory. |
| `/faces` | **Core CLI.** Direct access to the faces platform: create faces, compile documents, import YouTube videos, chat, compare faces, compose with Face Math, manage API keys and billing. |

## The hierarchy

```
Individual face     →  FACE.md   →  ~/.faces/catalog/<alias>/
Group of faces      →  TEAM.md   →  ~/.faces/teams/<team-name>/
Workflow            →  SKILL.md  →  manyfaced-<skillname>/
```

A manyfaced skill's SKILL.md is a conductor's score. It says: "In step 1, use this solo face. In step 3, convene this team using their protocol. In step 5, use this other face." The SKILL.md doesn't contain cognitive depth (that's FACE.md) or collaboration logic (that's TEAM.md). It wires them together.

## The filesystem

```
~/.faces/
  catalog.json                    # auto-generated index
  catalog/                        # individual faces
    garry-tan/FACE.md
    skeptic/FACE.md
  teams/                          # groups + protocols
    advisory-board/TEAM.md
  skills/                         # auto-generated face slash commands
    face-garry-tan/SKILL.md
```

Everything under `~/.faces/` is the Faces ecosystem. Catalog, teams, and face-skills all compound independently across projects and sessions.

## FACE.md format

Every face has a living document:

```yaml
---
name: garry-tan
description: YC president, builder-investor hybrid
role: startup-advisor
source_type: public-figure
tags: [startup, product, builder, investor]
compiled_tokens: 47200
sources_compiled: 6
formula: null
---
```

```markdown
## Queued
- [ ] https://youtube.com/watch?v=abc — Stanford lecture on fundraising
- [x] https://en.wikipedia.org/wiki/Garry_Tan *(compiled 2026-03-28)*

## Sources
- Wikipedia bio — career trajectory, YC timeline (12.4k tokens)
- "Unconventional Advice" talk — core philosophy (18.2k tokens)

## Lessons
- Defaults to Silicon Valley framing. Needs grounding for non-tech contexts.
- Users respond best when this face pushes back on premises before advising.

## Notes
- Consider adding blog posts for written voice (currently only video/spoken)
```

The FACE.md grows with the face. Queued tracks what to compile next. Sources records what's been compiled. Lessons captures what works and what doesn't. Notes is a freeform scratchpad. It's not a spec — it's the face's living file.

## Troubleshooting

**`faces: command not found`** — Run `npm install -g faces-cli`

**Skills not showing up?** — Make sure you cloned to `~/.claude/skills/faces-skill` (not `~/.claude/skills/faces-skill/faces-skill`)

**`401 Unauthorized`** — Run `faces auth:login --email you@example.com --password 'your-password'` or check `faces config:show`

**Never run `faces config:clear`** — It wipes all credentials with no recovery. If auth fails, fix the specific value with `faces config:set`.

---

[faces.sh](https://faces.sh) · [docs](https://docs.faces.sh) · [CLI reference](faces/references/REFERENCE.md)
