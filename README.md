# verse-uefn-skill

Pi agent skill for **Epic Games' Verse** programming language and **UEFN** (Unreal Editor for Fortnite) development. Covers Verse syntax, types, failable expressions, effects, concurrency, `creative_device`/NPC patterns, multiplayer, persistence, and `@editable` properties.

## Installation

### Pi (recommended)

```bash
# From npm (once published)
pi install npm:@vl4dt/verse-uefn-skill

# From git (works now)
pi install git:github.com/vl4dt/verse-uefn-skill@v1.0.0

# From a local clone
git clone https://github.com/vl4dt/verse-uefn-skill.git && cd verse-uefn-skill
pi install .
```

### Claude Code

```bash
git clone https://github.com/vl4dt/verse-uefn-skill.git ~/verse-uefn-skill
mkdir -p ~/.claude/skills
ln -s ~/verse-uefn-skill/skills/verse-uefn ~/.claude/skills/verse-uefn
```

### Manual (any agent)

```bash
git clone https://github.com/vl4dt/verse-uefn-skill.git
# Copy skills/verse-uefn/ into your agent's skills directory
```

## What you get

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — language cheat sheet, workflows, best practices |
| `REFERENCE.md` | Comprehensive Verse language reference (types, effects, failure, concurrency, patterns) |
| `EXAMPLES.md` | 12 working code examples (devices, game manager, scoring, rounds, inventory, NPC, etc.) |

## Skill contents

- **Verse language fundamentals** — immutability, mutability, type inference, failable expressions
- **Effect system** — `<computes>`, `<reads>`, `<writes>`, `<transacts>`, `<decides>`, `<suspends>`
- **Failure & speculative execution** — first-class failure control flow with auto-rollback
- **Structured concurrency** — `sync:`, `race:`, `rush:`, `branch:`
- **UEFN device development** — `creative_device`, `@editable` properties, lifecycle hooks
- **Multiplayer patterns** — player lifecycle, safe references, server/client prediction
- **Persistence** — `<persistable>` types, save/load with versioning best practices
- **NPC behavior** — custom AI agents extending `creative_npc_agent`

## Contributing

Issues and PRs welcome at https://github.com/vl4dt/verse-uefn-skill

## License

MIT
