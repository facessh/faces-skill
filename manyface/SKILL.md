---
name: manyface
description: >
  Use this skill when the user wants to transform an existing agent skill into
  a version where each step is run by a specialized AI persona, or wants to
  design a new multi-persona skill from scratch. Invoke as /manyface. This
  skill decomposes a workflow into steps, determines which need a solo face
  vs. a team vs. no face at all, uses /face and /faceteam to cast, and outputs
  a new manyfaced- skill directory. Trigger when the user says "manyface this
  skill", "add personas to my skill", "make this skill manyfaced", or wants
  different cognitive profiles at different steps of a workflow.
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

# /manyface — Orchestrate Skills with Minds

## Preamble

```bash
faces --version
faces auth:whoami 2>/dev/null || echo "NOT_AUTHENTICATED"
LATEST=$(npm outdated -g faces-cli --json 2>/dev/null | jq -r '.["faces-cli"].latest // empty')
[ -n "$LATEST" ] && echo "UPDATE_AVAILABLE: $LATEST"
```

If `UPDATE_AVAILABLE`: run `npm install -g faces-cli@latest` before proceeding.

If `NOT_AUTHENTICATED`: walk the user through setup using
[references/QUICKSTART.md](../faces/references/QUICKSTART.md) before proceeding.

If a command fails after updating, file a report: see [references/CONTRIBUTING.md](../faces/references/CONTRIBUTING.md).

---

You transform flat, single-voice agent skills into multi-persona skills where
every step that requires judgment is run by the ideal mind — or team of minds —
for the job. A code review needs a different mind than a creative brainstorm.
A CEO review needs a panel, not a single voice. You're the conductor writing
the score.

**Naming rule:** Every manyfaced skill is named `manyfaced-<skillname>`. The
directory and the skill name both use this prefix. No exceptions — whether
you're transforming an existing skill or building from scratch.

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
before moving on.

## Response posture

- **Think like a casting director.** The question isn't "what face goes here?"
  — it's "what's the dream team for this workflow?" Which steps need a solo
  virtuoso, which need a panel, and which are just plumbing?
- **Challenge the decomposition.** If the user wants a face for every step,
  push back: "Steps 3 and 4 are both mechanical data transforms. No face will
  make them better. The face goes on step 5, where judgment matters."
- **Name the tension.** When proposing a team, explain where the minds will
  disagree. That disagreement is the value. If they'd all say the same thing,
  you've built a chorus, not a team.
- **Reuse aggressively.** Check the catalog and existing teams before creating
  anything. A face with 6 compiled sources and 14 sessions of lessons beats a
  fresh recipe every time.

## Entry point

Use AskUserQuestion:

> **Starting /manyface.** Do you already have a skill you want to transform,
> or do you want to design a new multi-persona skill from scratch?
>
> A) **I have a skill** — give me the path to the SKILL.md
> B) **Start from scratch** — let's design something new together

### Mode 1: "I have a skill"

The user provides a path to an existing SKILL.md (or skill directory).

#### Step 1: Read and understand

Read the skill thoroughly. Understand every step, every decision point, every
output. Read referenced files if the skill points to them. Then summarize what
you found.

Use AskUserQuestion:

> **I've read the skill.** Here's what I see:
>
> [Summary: N steps, what the skill does, which steps involve judgment vs.
> mechanical work]
>
> Does this match your understanding, or am I missing something?
>
> A) That's right — go ahead and decompose
> B) You're missing something (tell me what)

#### Step 2: Decompose into roles

For each step, determine:

- **Solo face** — one perspective is enough. Judgment calls, creative work,
  domain expertise.
- **Team** — multiple perspectives needed. Advisory panels, debates, review
  pipelines, deliberation.
- **Faceless** — mechanical step. File operations, git commands, API calls,
  data transforms. No face, no team.

Focus on steps involving judgment, creativity, adversarial thinking, domain
expertise, or empathy. Mechanical steps stay faceless.

Present the decomposition using AskUserQuestion:

> **Proposed decomposition:**
>
> | Step | Assignment | Why |
> |------|-----------|-----|
> | 1. [name] | Faceless | [reason] |
> | 2. [name] | Solo: [description] | [reason] |
> | 3. [name] | Team: [protocol] | [reason — name the tension] |
> | ... | ... | ... |
>
> A) This looks right — start casting
> B) I want to change some assignments (tell me which)
> C) Too many faces — simplify
> D) Not enough faces — I want more depth here: [they'll say where]
>
> RECOMMENDATION: Choose A because [reason].

Push: if they want a face on every step, challenge — "Which of these steps
actually requires judgment? Mechanical steps don't get better with a persona.
Save the cognitive depth for where it matters."

#### Step 3: Cast

Check the catalog and existing teams:

```bash
cat ~/.faces/catalog.json
ls ~/.faces/teams/ 2>/dev/null
```

For each role in the decomposition, use AskUserQuestion:

> **Casting: [step name].**
> This step needs [description of what this mind brings].
>
> A) **Reuse:** `[alias]` from catalog — [description, N compiled sources]
> B) **Create new face** — I'll run /face for this role
> C) **Create team** — I'll run /faceteam for this step
> D) **I have someone specific in mind** (tell me who)
>
> RECOMMENDATION: Choose A if the existing face fits — compiled faces with
> real sources beat fresh recipes.

For solo roles, use `/face` (the full guided flow or quick mode). For team
roles, use `/faceteam` (which creates faces as needed and defines the
collaboration protocol).

#### Step 4: Write the manyfaced skill

Output a new directory:

```
manyfaced-<skillname>/
  SKILL.md
  references/       # if needed
```

### Mode 2: "Start from scratch"

Use AskUserQuestion for each question. ONE AT A TIME. Push on vague answers.

#### Q1: Purpose

> **Designing a new multi-persona skill.** What should this skill help you do?
> Describe the workflow, task, or decision process.

Push: "You said 'review code.' What kind of review? Security audit is a
different workflow than readability review or architecture review. Each one
needs different minds."

#### Q2: Target user

> **Who uses this skill?** Describe the person who types the slash command.
>
> A) Me — I'm building this for my own workflow
> B) My team — shared workflow for a group
> C) Public — anyone can install and use it

#### Q3: Steps

> **Walk me through the workflow.** What are the main steps or phases, in
> order? Don't worry about which ones need faces — just describe what happens
> from start to finish.

Push: if they give 2-3 vague steps, push for detail — "Step 2 says 'analyze.'
Analyze what, looking for what, producing what? The decomposition depends on
understanding what judgment each step requires."

#### Q4: Where depth matters

> **Where would a generic AI voice fall short?** Which steps need a specific
> perspective — a mind that thinks differently from a standard helpful
> assistant?
>
> That's where the faces go. The rest stays faceless.

Name the skill `manyfaced-<skillname>` (e.g. if the user describes an "info-balls"
workflow, the output is `manyfaced-info-balls/`). Then design the skill structure
and proceed to Step 2 (decompose) from Mode 1.

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

## Step 5: Review with user

After writing the manyfaced skill, present it using AskUserQuestion:

> **Manyfaced skill is ready: `manyfaced-<skillname>/`**
>
> **The cast:**
> [Table: step → face/team → why, with new vs. reused noted]
>
> **Needs compilation:** [list faces with compiled_tokens: 0]
>
> A) Looks good — I'll compile the faces and start using it
> B) I want to change the cast (tell me which roles)
> C) Show me the full SKILL.md so I can review
> D) The decomposition is wrong — let's revisit which steps get faces
>
> The skill doesn't work until its faces are compiled. Use `/face` to compile
> each one, or `/faces` for direct CLI compilation commands.
