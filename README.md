# faces-skill

> Every AI agent sounds the same. Helpful. Direct. Forgettable. **Faceless.**

The Faces Platform compiles source texts — documents, interviews, YouTube talks — into cognitive primitives that change how an LLM composes language. Not a character description. Not a system prompt. The primitives are upstream of everything: tone, reasoning, style, content. They determine what word comes next.

The result is a face that specializes an LLM with perspective, coherence, and nuance that doesn't collapse back into LLM-speak on long threads — using far fewer tokens than prompt-stuffing approaches like SOUL.md files.

**faces-skill** is a bundle of five slash commands that turn Claude Code (or OpenClaw) into a face creation and orchestration system:

- **`/face`** — Create a face from scratch. Interview, research, compile.
- **`/facechat`** — Chat with any face in your catalog. Pass an alias or browse to pick one.
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
Please install and setup Faces:
1. clone https://github.com/faces-sh/faces-skill.git into ~/.claude/skills/faces-skill then inspect the setup script and run it if it looks safe
2. ask if I already have a Faces account, and if so walk me through getting you access, else tell me about account options and help me get one
3. tell me about the main Faces skills - /face, /facechat, /faceteam, /manyface - then offer to restart the session so the slash commands take effect
```

Use this for **OpenClaw**:

```
Please install Faces:
1. clawhub install faces
2. run QUICKSTART.md exactly
3. tell me how to use it
```

## Community catalog

Browse and install manyfaced skills built by the community:
[**faces-sh/manyfaced**](https://github.com/faces-sh/manyfaced)

Built a manyfaced skill? Run `/manyface` and choose "publish" at the end to
prep a PR to the catalog.

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
git clone --depth 1 https://github.com/faces-sh/faces-skill.git ~/.claude/skills/faces-skill
```

Then run the setup script (it symlinks each skill into `~/.claude/skills/` so
Claude Code discovers them):

```bash
cd ~/.claude/skills/faces-skill && ./setup
```

You now have `/face`, `/facechat`, `/faceteam`, `/manyface`, and `/faces` available as slash commands.

### OpenClaw

```bash
git clone --depth 1 https://github.com/faces-sh/faces-skill.git ~/.openclaw/workspace/skills/faces-skill
cd ~/.openclaw/workspace/skills/faces-skill && ./setup
```

## See it work

### Create a face

```
You:    /face "Albert Einstein"
Claude: [searches for Einstein — finds Wikipedia, letters, lectures, interviews]
        [checks catalog — no existing match]
        [creates face on platform, writes FACE.md recipe]

        Recipe ready at ~/.faces/catalog/einstein/FACE.md
        Sources queued:
        - [ ] Wikipedia bio — career arc, key breakthroughs, philosophy of science
        - [ ] "On the Electrodynamics of Moving Bodies" — original reasoning style
        - [ ] 1950 Scientific American essay — late-career reflections on physics
        - [ ] Einstein-Besso letters — informal voice, how he thinks through problems
        Review and edit, then I'll compile.

You:    Looks good, compile it.
Claude: [imports Wikipedia bio, Scientific American essay, Electrodynamics paper,
         Einstein-Besso letters — all queued for compilation]
        [compiles Wikipedia bio — 14.1k tokens, 38 primitives]
        [compiles Electrodynamics paper — 11.2k tokens, 31 primitives]
        [compiles Scientific American essay — 9.8k tokens, 27 primitives]
        [compiles Einstein-Besso letters — 16.4k tokens, 44 primitives]
        Done. 140 cognitive primitives across 4 sources.
        Chat with him anytime: /facechat einstein
```

### Chat with a face

```
You:    /facechat einstein "I'm stuck. I have two competing models and the data fits both."
Einstein: That is not a problem — that is a gift. When two theories
          explain the same facts, you do not yet understand what you
          are really measuring. The question is not which model is
          correct. The question is: what experiment would make them
          disagree? Find the disagreement and you find the physics...

You:    /facechat
Claude: Your catalog:

        Faces:
        1. einstein — theoretical physicist, first-principles thinker
        2. skeptic — adversarial critic, assumption challenger

        Teams:
        3. research-review-panel — round-robin critique panel

        Who do you want to chat with?

You:    2
Claude: Chatting with skeptic. What would you like to ask or discuss?

You:    Is my startup idea defensible?
Skeptic: Define "defensible." You mean a moat? Because if your answer
         is "we'll move faster," that's not a moat — that's a prayer...
```

### Build a team

```
You:    /faceteam "I need a panel to critique my research paper"
Claude: [checks catalog — finds einstein, creates 2 new faces]
        [designs round-robin protocol as a mermaid diagram]

        Team: research-review-panel (3 rounds max)

        graph TD
            Q[Query] --> E(einstein)
            Q --> M(methodology)
            Q --> S(skeptic)
            E --> M
            M --> S
            S --> CHK{All 3 converged on recommendation, or round = 3}
            E --> CHK
            M --> CHK
            CHK -->|next round| E
            CHK -->|done| OUT[Response]

        Three shapes: (face) gets called, [instruction] agent executes,
        {condition} agent branches. Edges show who sees what.

        TEAM.md saved to ~/.faces/teams/research-review-panel/
```

### Chat with a team

```
You:    /facechat research-review-panel "Here's my draft paper on cognitive primitives..."
Claude: [reads TEAM.md, walks the mermaid protocol]

        einstein: The central claim — that cognitive primitives are
                  extractable from text — needs a falsifiability criterion.
                  What observation would disprove this?...

        methodology: The sample size in Section 4 is too small to support
                     the generalization in your conclusion. You need at
                     minimum...

        skeptic: You're assuming primitives are stable across contexts.
                 What if the same source material produces different
                 primitives under different LLMs?...

        [Consensus check: all 3 flagged Section 4 methodology — converged.
         Einstein and skeptic diverge on falsifiability framing — round 2.]

        ...
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
| `/face` | **Guided face creation.** Six steps: interview → research (real URLs via web search) → sketch FACE.md → iterate with user → compile. Quick mode: `/face "Person Name"` skips the interview. |
| `/facechat` | **Chat with any face or team.** `/facechat einstein` to chat with a face, `/facechat review-panel` to run a team's protocol, or `/facechat` to browse your catalog and pick. Supports model override and template references. |
| `/faceteam` | **Team composition.** Composes faces into a team with a self-sufficient mermaid protocol diagram. Three shapes: `(face)` rounded rects get called, `[instruction]` sharp rects the agent executes, `{condition}` diamonds the agent branches on. Produces a TEAM.md in `~/.faces/teams/`. |
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
    einstein/FACE.md
    skeptic/FACE.md
  teams/                          # groups + protocols
    advisory-board/TEAM.md
```

Everything under `~/.faces/` is the Faces ecosystem. Catalog and teams compound independently across projects and sessions. Chat with any face via `/facechat`.

## FACE.md format

Every face has a living document:

```yaml
---
name: albert-einstein
description: Theoretical physicist, first-principles thinker
role: scientific-advisor
source_type: public-figure
tags: [physics, science, philosophy, first-principles]
compiled_tokens: 47200
sources_compiled: 6
formula: null
---
```

```markdown
## Queued
- [ ] https://youtube.com/watch?v=abc — 1950 Scientific American essay
- [x] https://en.wikipedia.org/wiki/Albert_Einstein *(compiled 2026-03-28)*

## Sources
- Wikipedia bio — career arc, key breakthroughs, philosophy of science (14.1k tokens)
- Scientific American essay — late-career reflections on physics (9.8k tokens)

## Lessons
- Defaults to physics analogies. Needs grounding for non-scientific contexts.
- Users respond best when this face reframes the question before answering.

## Notes
- Consider adding Einstein-Besso letters for informal reasoning voice
```

The FACE.md grows with the face. Queued tracks what to compile next. Sources records what's been compiled. Lessons captures what works and what doesn't. Notes is a freeform scratchpad. It's not a spec — it's the face's living file.

## Troubleshooting

**`faces: command not found`** — Run `npm install -g faces-cli`

**Skills not showing up?** — Make sure you cloned to `~/.claude/skills/faces-skill` (not `~/.claude/skills/faces-skill/faces-skill`)

**`401 Unauthorized`** — Run `faces auth:login --email you@example.com --password 'your-password'` or check `faces config:show`

**Never run `faces config:clear`** — It wipes all credentials with no recovery. If auth fails, fix the specific value with `faces config:set`.

---

[faces.sh](https://faces.sh) · [docs](https://docs.faces.sh) · [CLI reference](faces/references/REFERENCE.md) · [manyfaced catalog](https://github.com/faces-sh/manyfaced)
