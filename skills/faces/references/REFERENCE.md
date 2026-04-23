# Full command reference

```
faces auth:login        --email  --password
faces auth:logout
faces auth:register     --email  --password  --username  [--plan free|connect]  [--invite-key]
faces auth:whoami
faces auth:refresh
faces auth:connect      <provider>  [--manual]
faces auth:disconnect   <provider>
faces auth:connections

faces face:create       --name  --alias  [--default-model MODEL]  [--description TEXT]  [--tag TAG...]  [--formula EXPR | --attr KEY=VALUE... --tool NAME...]
faces face:list         [--tag TAG...]
faces face:get          <alias>
faces face:attributes
faces face:update       <alias>  [--name]  [--default-model MODEL]  [--description TEXT]  [--tag TAG...]  [--formula EXPR]  [--attr KEY=VALUE]...
faces face:delete       <alias>  [--yes]
faces face:stats
faces face:upload       <alias>  --file PATH  [--kind document|thread]  [--perspective first-person|third-person]  [--face-speaker NAME]
faces face:diff         --face ALIAS  --face ALIAS  [--face ALIAS]...
faces face:neighbors    <alias>  [--k N]  [--component face|beta|delta|epsilon]  [--direction nearest|furthest]

faces face:tag:list     <alias>
faces face:tag:add      <alias>  --tag TAG  [--tag TAG...]
faces face:tag:set      <alias>  --tag TAG  [--tag TAG...]
faces face:tag:remove   <alias>  <tag>

faces chat:chat         <face_alias>  -m MSG  [--llm MODEL]  [--system]  [--stream]
                        [--max-tokens N]  [--temperature F]  [--file PATH]  [--responses]  [--oauth-only]
faces chat:messages     <face@model | model>  -m MSG  [--system]  [--stream]  [--max-tokens N]  [--oauth-only]
faces chat:responses    <face@model | model>  -m MSG  [--instructions]  [--stream]  [--oauth-only]

faces compile:import       <alias>  --url YOUTUBE_URL  [--type document|thread]  [--perspective first-person|third-person]  [--face-speaker LABEL]  [--no-wait]
faces compile:upload       <alias>  --file PATH  [--kind document|thread]  [--perspective first-person|third-person]  [--face-speaker NAME]  [--no-wait]

faces compile:doc          <alias>  (--content TEXT | --file PATH)  [--label]  [--perspective first-person|third-person]  [--timeout N]  [--no-wait]
faces compile:doc:create   <alias>  [--label]  (--content TEXT | --file PATH)  [--perspective first-person|third-person]
faces compile:doc:make     <doc_id>  [--timeout N]  [--no-wait]
faces compile:doc:pause    <doc_id>
faces compile:doc:reset    <doc_id>
faces compile:doc:list     <alias>
faces compile:doc:get      <doc_id>
faces compile:doc:edit     <doc_id>  [--label]  [--content TEXT | --file PATH]  [--perspective first-person|third-person]
faces compile:doc:delete   <doc_id>

faces compile:thread:create   <alias>  [--label]  [--oauth-only]
faces compile:thread:list     <alias>
faces compile:thread:get      <thread_id>
faces compile:thread:edit     <thread_id>  [--label TEXT]  [--face-speaker NAME]
faces compile:thread:message  <thread_id>  -m MSG  [--oauth-only]
faces compile:thread:make     <thread_id>  [--timeout N]  [--no-wait]
faces compile:thread:pause    <thread_id>
faces compile:thread:reset    <thread_id>
faces compile:thread:sync     <thread_id>
faces compile:thread:delete   <thread_id>  [--yes]

faces catalog:doctor      [--fix]  [--generate]
faces catalog:list
faces catalog:backup
faces catalog:restore     [FILE]  [--compile]
faces catalog:manyfaced   [--skill NAME]  [--install NAME --skills-dir PATH]  [--refresh]

faces compile:all         [--timeout N]

faces keys:create   --name  [--expires-days N]  [--budget F]  [--face ALIAS]...  [--model NAME]...
faces keys:list
faces keys:revoke   <key_id>  [--yes]
faces keys:update   <key_id>  [--name]  [--budget F]  [--reset-spent]

faces billing:balance
faces billing:subscription
faces billing:quota
faces billing:usage      [--group-by api_key|model|llm|date]  [--from DATE]  [--to DATE]
faces billing:topup      --amount F  [--payment-ref REF]
faces billing:checkout   [--plan connect]
faces billing:card-setup
faces billing:llm-costs  [--provider openai|anthropic|...]

faces team:create       --name  [--description TEXT]  [--protocol TEXT]  [--protocol-file PATH]  [--tag TAG...]
faces team:list
faces team:get          <team_id>
faces team:update       <team_id>  [--name]  [--description TEXT]  [--protocol TEXT]  [--protocol-file PATH]
faces team:delete       <team_id>  [--yes]
faces team:members      <team_id>
faces team:add          <team_id>  --face ALIAS  [--face ALIAS...]
faces team:remove       <team_id>  <alias>

faces team:tag:list     <team_id>
faces team:tag:add      <team_id>  --tag TAG  [--tag TAG...]
faces team:tag:set      <team_id>  --tag TAG  [--tag TAG...]
faces team:tag:remove   <team_id>  <tag>

faces account:state
faces account:preferences [KEY] [VALUE]

faces config:set    <key> <value>
faces config:show
faces config:clear  [--yes]
```

## Default model (`--default-model`)

The `--default-model` flag on `face:create` and `face:update` sets the LLM used when no `--llm` override is provided to `chat:chat`. Without a default model, the face inherits the user's account-level default model (set via `account:preferences`).

```bash
faces face:create --name "Ada" --alias ada --default-model gpt-5.4-mini
faces chat:chat ada -m "hello"    # uses gpt-5.4-mini automatically
```

The user's account default model can be viewed and changed with:

```bash
faces account:preferences                            # show current preferences
faces account:preferences default_model gpt-5.4      # set default model
```

New faces inherit the account default model at creation time (one-time copy, not a live link). Changing the account default does not update existing faces.

## Chat auto-routing

`chat:chat` automatically routes to the correct API endpoint based on the model provider:

- `claude-*` models → Anthropic Messages API (`/v1/messages`)
- All other models → OpenAI Chat Completions API (`/v1/chat/completions`)
- `--responses` flag → OpenAI Responses API (`/v1/responses`)

The backend may auto-route certain models (e.g. `gpt-5.4`) from Chat Completions to the Responses API internally. The CLI detects this via the `X-Faces-Routed-Endpoint` response header and parses the response body in the correct format automatically. In `--json` mode, the response includes a `_meta` block with `provider`, `cost_usd`, and `routed_endpoint` fields.

`chat:messages` and `chat:responses` are still available for direct endpoint access.

## OAuth-only mode (`--oauth-only`) — Subscription Connect only

The `--oauth-only` flag and `api_fallback` preference only apply to **Subscription Connect** users who have linked their ChatGPT account via `faces auth:connect openai`. Pay-per-token users do not use OAuth and these settings have no effect for them.

The `--oauth-only` flag prevents fallback to paid system keys. When set, requests that fail OAuth return a 422 error instead of silently falling back to credits. Available on `chat:chat`, `chat:messages`, `chat:responses`, `compile:thread:create`, and `compile:thread:message`.

The account-level equivalent is the `api_fallback` preference. When `api_fallback` is `false` (default for Subscription Connect), all requests behave as if `--oauth-only` is set — OAuth failures return 422 instead of falling back to paid keys. Set it to `true` to allow automatic paid fallback when OAuth fails.

```bash
faces account:preferences api_fallback false    # default: no fallback (Subscription Connect)
faces account:preferences api_fallback true     # allow paid fallback when OAuth fails
```

## Account preferences

Server-side preferences stored on the user's account. Viewed and modified with `account:preferences`:

| Key | Values | Default | Effect |
|---|---|---|---|
| `api_fallback` | `true` / `false` | `false` | When false, OAuth failures return 422 instead of falling back to paid system keys |
| `default_model` | any valid model | `gpt-5.4` | Model inherited by new faces that don't specify `--default-model` |

## Face description (`--description`)

The `--description` flag on `face:create` and `face:update` sets a plain-text bio stored on the server (max 1500 chars). It is also synced to the local FACE.md catalog.

```bash
faces face:create --name "Ada" --alias ada --description "Theoretical physicist and first-principles thinker"
faces face:update ada --description "Updated bio"
```

`catalog:doctor --fix` pulls descriptions from the server. `catalog:doctor --generate` creates descriptions via LLM and syncs them back to the server.

## Face tags (`--tag`)

Lowercase string labels for organization and search. Max 64 chars, max 32 per face.

```bash
# Set tags on create
faces face:create --name "Ada" --alias ada --tag research --tag physics

# Replace all tags
faces face:tag:set ada --tag research --tag updated

# Add tags (append, idempotent)
faces face:tag:add ada --tag new-tag

# Remove a tag
faces face:tag:remove ada old-tag

# List tags
faces face:tag:list ada

# Filter face list by tags (AND logic)
faces face:list --tag research --tag physics
```

## Teams

Named groups of faces with optional description, protocol (mermaid diagram), and tags.

```bash
# Create a team
faces team:create --name "Review Panel" --description "Research critique panel" --tag review

# Add faces
faces team:add TEAM_ID --face alice --face bob

# List teams (personal Guild is filtered out)
faces team:list

# Set protocol (mermaid diagram)
faces team:update TEAM_ID --protocol-file workflow.mmd

# Manage members
faces team:members TEAM_ID
faces team:remove TEAM_ID alice

# Team tags (same pattern as face tags)
faces team:tag:add TEAM_ID --tag urgent
faces team:tag:list TEAM_ID
```

Teams also have a local TEAM.md representation at `~/.faces/teams/<name>/TEAM.md` with YAML frontmatter (name, description, tags, members as aliases) and the protocol as the markdown body.

## Compiling documents (`compile:doc`)

`compile:doc` is the recommended one-step command for compiling a document into a face. Pass the face alias as the first argument. It handles create → compile (prepare + sync) automatically with real-time progress:

```bash
faces compile:doc alice --file essay.txt
```

Output:
```
Creating document... done (abc123)
Compiling (3 chunks):
  [1/3] ε=3 β=2 δ=1 α=5
  [2/3] ε=5 β=6 δ=2 α=12
  [3/3] ε=8 β=6 δ=5 α=26 (syncing)
Done.
```

For an already-created document, use `compile:doc:make <doc_id>` to compile it.

`--timeout` sets the polling timeout in seconds (default: 600 / 10 minutes).

## Deleting sources

`compile:thread:delete` and `compile:doc:delete` are clean — they remove all
cognitive components (beta, delta, alpha, epsilon) extracted from that source.
The face's profile and component counts update immediately. No re-sync needed.

Do NOT warn users that components may persist after deletion — they don't.

## Catalog

The CLI maintains a local catalog at `~/.faces/catalog/` with a `FACE.md` file per face (YAML frontmatter + markdown notes) and a consolidated `~/.faces/catalog.json` index. The catalog is managed automatically on `face:create`, `face:update`, and `face:delete`.

- `faces catalog:doctor` — diagnose missing, stale, or orphaned catalog entries
- `faces catalog:doctor --fix` — rebuild catalog from API (syncs descriptions from server)
- `faces catalog:doctor --generate` — fix + generate descriptions via LLM (syncs back to server)
- `faces catalog:backup` — snapshot all faces (with descriptions, tags), teams, documents, and threads to `~/.faces/backups/<timestamp>.json`
- `faces catalog:restore [FILE]` — restore faces, tags, teams, and source material from a backup (defaults to most recent). `--compile` runs `compile:all` after upload.
- `faces catalog:list` — print catalog contents
- `faces compile:all` — compile all uncompiled documents and threads across all faces (one at a time with progress)
- `faces config:set catalog false` — disable catalog management
- `faces config:set catalog_model gpt-5-nano` — set the model used for description generation

## Attributes (`--attr`)

The `--attr KEY=VALUE` flag on `face:create` and `face:update` sets basic demographic facts on a Face directly, without compiling a document. The flag is repeatable — pass one `--attr` per fact. These facts anchor the persona and improve compilation quality when source material is added later.

**Common attribute keys:**

| Key | Example value |
|---|---|
| `gender` | `female`, `male`, `non-binary` |
| `age` | `34` |
| `location` | `Portland, OR` |
| `occupation` | `nurse practitioner` |
| `education_level` | `master's degree` |
| `religion` | `Buddhist` |
| `ethnicity` | `Korean American` |
| `nationality` | `American` |
| `marital_status` | `married` |

These are the most common keys. Many more are accepted across categories including birth details, family, career, education, housing, and immigration. **Only keys on the accepted list work — unsupported keys are dropped with a warning in the response.** Run `faces face:attributes` for the complete categorized list.

**Example — create a Face with basic facts:**
```bash
faces face:create --name "Maria Chen" --alias maria \
  --attr gender=female --attr age=34 \
  --attr location="Portland, OR" \
  --attr occupation="nurse practitioner" \
  --attr education_level="master's degree" \
  --attr marital_status=married
```

**Example — add facts to an existing Face:**
```bash
faces face:update maria --attr religion=Buddhist --attr ethnicity="Korean American"
```

> Note: `--attr` cannot be combined with `--formula`. Composite faces inherit their facts from their component faces.

## Global flags

Any command accepts these flags:

```
faces [--base-url URL] [--token JWT] [--api-key KEY] [--json] COMMAND
```

## Environment variables

| Variable | Purpose |
|---|---|
| `FACES_BASE_URL` | Override API base URL (default: `api.faces.sh`) |
| `FACES_TOKEN` | JWT authentication token |
| `FACES_API_KEY` | API key authentication |
