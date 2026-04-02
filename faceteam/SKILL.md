---
name: faceteam
description: >
  Use this skill when the user needs multiple AI faces working together on a
  task — an advisory panel, a debate, a review pipeline, or any collaboration
  that requires more than one perspective. Invoke as /faceteam. This skill
  composes faces into a team with a defined collaboration protocol, produces a
  TEAM.md file with a mermaid flowchart diagram showing exactly how the faces
  interact, and stores it in ~/.faces/teams/. Uses /face to create any new
  faces needed. Trigger when the user says "build me a team", "I need a panel",
  "set up a debate", "create a review board", or describes a task needing
  multiple perspectives in concert.
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

# /faceteam — Compose Minds into Teams

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

You compose faces into teams with defined collaboration protocols. A single
face brings depth. A team brings depth AND tension — the skeptic who challenges
the optimist, the builder who grounds the visionary, the domain expert who
catches what generalists miss. Three well-chosen faces with genuine cognitive
diversity beat eight variations of "helpful expert."

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

- **Push for specificity.** "I need a team to review things" is too vague.
  What things? What kind of review? What goes wrong when a single reviewer
  handles it alone?
- **Challenge headcount.** If the user asks for 5 people, ask why 3 won't do.
  Cognitive diversity matters more than quantity. Every face on the team should
  bring a perspective the others can't.
- **Name the tension.** The best teams have productive disagreement built in.
  If all three faces would say the same thing, you've built a chorus, not a
  team. Identify where the perspectives will clash — that's the signal.
- **Recommend a protocol.** Don't ask the user to pick from five options they
  don't understand. Listen to what they need and recommend the right one. Then
  explain why.

## The flow

### Step 1: Understand the task

Use AskUserQuestion:

> **Building a team.** What does this team need to accomplish? Describe the
> task, decision, or workflow they'll handle.
>
> A few examples to calibrate:
> - "Evaluate whether we should pivot" → advisory panel (round robin)
> - "Review every PR before it ships" → review pipeline
> - "Stress-test my pitch from both sides" → debate
> - "Get independent opinions without groupthink" → voting

Wait for the answer. Then follow up with AskUserQuestion:

> **Understanding the collaboration.** Based on what you described, here's what
> I'm thinking:
>
> [Your analysis: how many faces, what roles, what protocol, and why. Name
> where the perspectives will clash — that's where the value is.]
>
> A) That sounds right — build it
> B) I want to adjust the roles or number of faces
> C) I had a different collaboration style in mind
>
> RECOMMENDATION: Choose A because [reason based on their task].

Push: if they want 5+ faces, challenge — "What does the 5th face say that the
other 4 don't? Every face on the team should earn its seat."

### Step 2: Cast the team

Check the catalog and existing teams first:

```bash
cat ~/.faces/catalog.json
ls ~/.faces/teams/ 2>/dev/null
```

For each role, use AskUserQuestion:

> **Casting role [N]: [role name].** This seat needs [description of what
> this face brings].
>
> A) **Reuse existing face:** `[alias]` — [description from catalog]. Already
>    has [N] compiled sources.
> B) **Create a new face** — I'll run /face to build one for this role
> C) **I have someone specific in mind** (tell me who)
>
> RECOMMENDATION: Choose A if the existing face fits — a compiled face with
> real sources beats a fresh recipe.

If creating new faces, run `/face` for each one (the full guided flow or quick
mode as appropriate). The user should approve each cast member before you move
on.

### Step 3: Design the protocol

Based on what you learned in Step 1, recommend a protocol. The protocol
determines how the faces interact — and it's defined with a mermaid flowchart
so a human can see the pattern at a glance.

**Protocol types and when to use them:**

**Round robin** — faces take turns, building on each other's responses.
Each face sees all prior responses. Good for advisory panels, brainstorming,
iterative refinement.
```mermaid
graph LR
    Q[Query] --> A[face-a]
    A --> B[face-b]
    B --> C[face-c]
    C --> CHECK{Consensus?}
    CHECK -->|no, round < N| A
    CHECK -->|yes or max rounds| OUT[Output]
```

**Pipeline** — sequential chain, each face adds a layer. Output of one becomes
input to the next. Good for review processes, quality gates, progressive
refinement.
```mermaid
graph LR
    IN[Input] --> A[face-a]
    A --> B[face-b]
    B --> C[face-c]
    C --> OUT[Output]
```

**Chief of staff** — one face coordinates, delegates to specialists,
synthesizes their responses. Good for complex decisions requiring multiple
domains.
```mermaid
graph TD
    Q[Query] --> COS[chief-of-staff]
    COS --> S1[specialist-1]
    COS --> S2[specialist-2]
    COS --> S3[specialist-3]
    S1 --> COS
    S2 --> COS
    S3 --> COS
    COS --> OUT[Synthesis]
```

**Debate** — two sides argue, a judge decides. Good for evaluating
controversial decisions, stress-testing proposals.
```mermaid
graph TD
    Q[Query] --> PRO[advocate]
    Q --> CON[critic]
    PRO --> J[judge]
    CON --> J
    J --> OUT[Decision]
```

**Voting** — all faces respond independently, results are tallied. No face
sees the others' responses. Good for calibration, avoiding groupthink.
```mermaid
graph TD
    Q[Query] --> A[face-a]
    Q --> B[face-b]
    Q --> C[face-c]
    A --> T[Tally]
    B --> T
    C --> T
    T --> OUT[Result]
```

Present your recommended protocol using AskUserQuestion:

> **Protocol recommendation.** For [their task], I recommend **[protocol]**
> because [reason]. Here's how it works:
>
> [One-sentence description of the flow]
>
> A) Use this protocol
> B) I'd prefer a different one (tell me which)
> C) Explain the other options so I can compare

### Step 4: Write the TEAM.md

```bash
mkdir -p ~/.faces/teams/<team-name>
cat > ~/.faces/teams/<team-name>/TEAM.md << 'TEAM'
<full TEAM.md content>
TEAM
```

TEAM.md format:

```markdown
---
name: <team-name>
description: <what this team does>
protocol: <round-robin | pipeline | chief-of-staff | debate | voting>
faces: [alias-1, alias-2, alias-3]
max_rounds: 3
---

## Protocol

```mermaid
<flowchart matching the protocol type, using actual face aliases as node labels>
```

## Roles

| Face | Role | Evaluates |
|------|------|-----------|
| alias-1 | <role> | <what they focus on> |
| alias-2 | <role> | <what they focus on> |
| alias-3 | <role> | <what they focus on> |

## Rules

- <how each face sees prior responses>
- <constraints on each face's behavior>
- <what happens if no consensus / edge cases>
- <termination conditions>

## Notes

<casting rationale, team dynamics, where the tension is, etc.>
```

The mermaid diagram goes right after frontmatter, before prose. The diagram
shape IS the documentation — a human should see the collaboration pattern at a
glance from the shape alone. Use actual face aliases as node labels, not generic
"Face A" placeholders.

### Step 5: Review with user

Use AskUserQuestion:

> **Team `[team-name]` is ready.** [N] faces, [protocol] protocol.
>
> [Summary: who's on the team, what each face brings, where the productive
> tension is]
>
> Faces needing compilation: [list any with compiled_tokens: 0]
>
> A) Looks good — compile all the faces for me now
> B) I want to change the roster or protocol
> C) Show me the full TEAM.md so I can review the details
> D) Don't compile yet — I want to review the FACE.md recipes first
>
> RECOMMENDATION: Choose A — the team doesn't work until its faces are compiled.

### Step 6: Compile the team's faces

If the user chose A (or comes back after reviewing), compile each face that
has `compiled_tokens: 0`. For each face, follow the `/face` skill's Step 5
(compile) flow:

1. Read the face's FACE.md from `~/.faces/catalog/<alias>/FACE.md`
2. Walk through each Queued item using `--no-wait`:
   - YouTube: `faces compile:import <alias> --url "<url>" --type document --no-wait --json`
   - Local file: `faces compile:doc <alias> --file <path> --no-wait --json`
3. Fire all compiles in parallel — don't wait for one to finish before starting the next
4. Poll all at the end: `faces compile:doc:get ID --json | jq '{prepare_status}'`
5. Update each FACE.md: check boxes in Queued, add entries to Sources

Compile all team members before telling the user the team is ready. The goal
is a fully operational team at the end of `/faceteam`, not a set of recipes
the user has to finish themselves.
