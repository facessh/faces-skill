# Interviews

An interview is a Q&A conversation where someone answers questions as the face — their responses become the source material that gets compiled into cognitive primitives. Interviews are the most accessible way to build a face: no pre-existing documents needed.

There are two modes. Mode 2 (agent-as-interviewer) is recommended when using this skill because the agent has conversational context that produces better, more targeted questions.

## Mode 1: Built-in interviewer

The backend's AI generates the questions. Good for a guided, hands-off experience.

```bash
# 1. Start — returns thread_id and the AI's opening question
RESULT=$(faces compile:thread:create alias --label "Interview" --json)
THREAD_ID=$(echo "$RESULT" | jq -r '.thread_id // .id')
# Show the AI's question to the user

# 2. Loop — send user's answer, get next question
RESULT=$(faces compile:thread:message "$THREAD_ID" -m "user's answer" --json)
# Show the next question to the user
# Repeat until the user is done (minimum 3 responses)

# 3. Compile
faces compile:thread:make "$THREAD_ID" --no-wait --json
```

Every call to `compile:thread:message` stores the user's answer AND generates the next AI question. The transcript stays coherent because both sides live in the thread.

## Mode 2: Agent-as-interviewer (recommended)

The agent runs the interview itself — no API calls during the conversation. This produces better interviews because the agent can tailor questions to the user's goals, probe deeper on interesting answers, and skip irrelevant topics.

**Step 1: Conduct the interview.** Ask questions, collect answers, follow up. Normal conversation — no Faces API calls during this phase.

**Step 2: Format the transcript.** Save the conversation as a text file with `Speaker: message` format, one turn per line:

```
Agent: Tell me about yourself.
Johnny: I grew up in a small town in Alabama...
Agent: What shaped your worldview?
Johnny: Reading philosophy as a teenager changed everything.
```

**Step 3: Upload and compile.**

```bash
THREAD_ID=$(faces compile:upload alias --file transcript.txt --kind thread \
  --face-speaker "Johnny" --json | jq -r '.thread_id // .id')
faces compile:thread:make "$THREAD_ID" --no-wait --json
```

`--face-speaker` tells the backend which speaker IS the face (mapped to `role=user`). All other speakers become `role=assistant`. Only user messages are compiled. If `--face-speaker` is omitted, the backend fuzzy-matches speaker names against the face's display name and alias.

**Label format:** For text transcripts, use the speaker name from the file (e.g. `--face-speaker "Johnny"`). For audio/video transcription, speakers are labeled `A`, `B`, `C` — use the short label (e.g. `--face-speaker B`). Match is case-insensitive but exact.

## Tips

- **Multiple interviews**: Compile each separately into the same face. Each adds to the existing cognitive primitives.
- **Minimum depth**: Aim for at least 5-10 substantive exchanges. Longer interviews produce richer faces.
- **Resuming a built-in interview**: If a thread wasn't compiled, retrieve it with `faces compile:thread:get THREAD_ID --json` and continue with `compile:thread:message`.
- **Compilation is async**: `compile:thread:make` may take time for long transcripts. Poll with `faces compile:thread:get THREAD_ID --json | jq -r '.prepare_status'` if it seems stalled — re-run the make command if stuck on "preparing" for more than 10 minutes.
