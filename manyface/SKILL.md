---
name: manyface
description: >
  Use this skill when the user wants to transform an existing agent skill into
  a version where each step is run by a specialized AI persona, or wants to
  design a new multi-persona skill from scratch. Invoke as /manyface. This
  skill decomposes a workflow into steps, determines which need a solo face
  vs. a team vs. no face at all, uses /face and /team to cast, and outputs a
  new manyfaced- skill directory. Trigger when the user says "manyface this
  skill", "add personas to my skill", "make this skill manyfaced", or wants
  different cognitive profiles at different steps of a workflow.
---

# /manyface — Orchestrate Skills with Minds

## Preamble

```bash
faces auth:whoami 2>/dev/null || echo "NOT_AUTHENTICATED"
```

If NOT_AUTHENTICATED: walk the user through setup using
[references/QUICKSTART.md](../faces/references/QUICKSTART.md) before proceeding.

---

You transform flat, single-voice agent skills into multi-persona skills where
every step that requires judgment is run by the ideal mind — or team of minds —
for the job. A code review needs a different mind than a creative brainstorm.
A CEO review needs a panel, not a single voice. You're the conductor writing
the score.

## Entry point

Ask the user:

> Do you already have a skill you want to manyface, or do you want to start
> from scratch?

### Mode 1: "I have a skill"

The user provides a path to an existing SKILL.md (or skill directory).

1. **Read the skill thoroughly.** Understand every step, every decision point,
   every output. Read referenced files if the skill points to them.

2. **Decompose into roles.** For each step, determine:

   - **Solo face** — one perspective is enough. Judgment calls, creative work,
     domain expertise. Use `/face` to create.
   - **Team** — multiple perspectives needed. Advisory panels, debates, review
     pipelines, deliberation. Use `/team` to compose.
   - **Faceless** — mechanical step. File operations, git commands, API calls,
     data transforms. No face, no team. Leave as-is.

   Focus on steps involving judgment, creativity, adversarial thinking, domain
   expertise, or empathy. Mechanical steps stay faceless.

3. **Cast.** For solo roles, use `/face` (or follow the same process: research,
   sketch FACE.md, write to catalog). For team roles, use `/team` (which
   creates faces as needed and defines the collaboration protocol).

4. **Write the manyfaced skill.** Output a new directory:

```
manyfaced-<skillname>/
  SKILL.md
  references/       # if needed
```

### Mode 2: "Start from scratch"

Run a short interrogation:

- What should this skill help you do?
- Who is the target user?
- What are the main steps or phases?
- Where do you need depth — where would a generic AI voice fall short?

Then design the skill and proceed from step 2 above.

## How to write the manyfaced SKILL.md

The manyfaced SKILL.md is a conductor's score. It wires faces and teams to
steps but never contains cognitive depth itself (that's FACE.md) or
collaboration logic (that's TEAM.md).

**For solo face steps:**

```markdown
### Step N: <step name>

**Face:** `<alias>` — <why this mind>

<instructions for this step>

Use `faces chat:chat <alias> -m "<prompt>"` or reference via `${<alias>}`.
```

**For team steps:**

```markdown
### Step N: <step name>

**Team:** `<team-name>` — <why this team>

Run this step using the protocol defined in
`~/.faces/teams/<team-name>/TEAM.md`.

<instructions — how to feed input, what to do with output>
```

**For faceless steps:**

```markdown
### Step N: <step name>

<instructions — no face or team assignment>
```

Include a **Setup** section at the top:

```markdown
## Setup — Compile the cast

This skill requires the following faces and teams. Review each recipe, then
compile using the `/faces` skill.

### Solo Faces

| Role | Face | Recipe |
|------|------|--------|
| <role> | `<alias>` | `~/.faces/catalog/<alias>/FACE.md` |

### Teams

| Role | Team | Protocol | Faces |
|------|------|----------|-------|
| <role> | `<team-name>` | `~/.faces/teams/<team-name>/TEAM.md` | alias-1, alias-2, alias-3 |

Faces with `compiled_tokens: 0` need compilation before use. See each FACE.md's
Queued section for source material.
```

## Key principles

**Solo face vs. team.** If one perspective is enough, use a solo face. If the
value comes from tension between perspectives (a skeptic challenging an
optimist, a tester finding what a builder missed), use a team. Don't use a team
when a solo face will do — teams cost more tokens and time.

**Reuse over creation.** Check the catalog and existing teams before creating
new ones. A face compiled from 6 sources is worth more than a fresh recipe.
The catalog compounds across skills.

**The face does the thinking, the skill does the routing.** The manyfaced skill
decides who handles what. The face brings depth. The team brings collaboration
logic. The skill never describes how a persona should think.

**Mechanical steps stay faceless.** Not every step needs a mind. File I/O, git
operations, API calls, data transforms — leave them alone.

## After outputting the manyfaced skill

Tell the user:
1. The cast — which faces and teams, and why each was chosen
2. Which are new vs. reused from the catalog
3. Which faces need compilation before the skill works
4. How the teams collaborate (point to the mermaid diagrams in TEAM.md files)
