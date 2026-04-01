# TODO

## Bundle structure (current)

```
faces-skill/
├── faces/           → /faces          — core CLI operations, compilation, chat
├── face/            → /face           — guided face creation (interview → compile → slash command)
├── team/            → /team           — compose faces into teams with protocols
└── manyface/        → /manyface       — decompose skills into multi-persona versions
```

Install: clone to `~/.claude/skills/faces-skill` — Claude Code auto-discovers
all SKILL.md files in subdirectories.

## Open items

- [ ] Compile timeout heuristics: benchmark compile times by source token count
- [ ] Face-skill discovery: /face generates symlinks to ~/.claude/skills/ and
      ~/.openclaw/workspace/skills/ — verify this works in both runtimes
- [ ] Bootstrap catalog: ship pre-built FACE.md recipes for common archetypes
- [ ] Description optimization: rerun automated trigger eval loop for all 4 skills
- [ ] Team protocol execution: document how manyfaced skills translate mermaid
      diagrams into actual LLM orchestration steps
- [ ] Lesson accumulation: define how/when lessons get appended to FACE.md
