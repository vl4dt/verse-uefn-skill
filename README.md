# verse-uefn-skill

Pi agent skill for **Epic Games Verse** and **UEFN** (Unreal Editor for Fortnite) development. It is designed to guide agents that work with VS Code, Epic's Verse extension, and the UEFN MCP server.

## Installation

### Pi (recommended)

```bash
# From npm (once published)
pi install npm:@vl4dt/verse-uefn-skill

# From git (works now)
pi install git:github.com/vl4dt/verse-uefn-skill@v1.1.0

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
| `SKILL.md` | Main skill instructions: UEFN workflow, MCP tool map, Verse/Python boundaries |
| `REFERENCE.md` | Comprehensive Verse language reference (types, effects, failure, concurrency, patterns) |
| `EXAMPLES.md` | Conservative examples and MCP-assisted edit loop |

## Skill contents

- **UEFN agent workflow**: inspect project context, edit narrowly, validate with diagnostics
- **MCP integration**: `uefn_get_project_context`, Verse file tools, diagnostics, editor state
- **Verse/Python boundary**: Verse for runtime gameplay, Python for editor automation
- **Verse fundamentals**: immutability, mutability, failable expressions, effects, concurrency
- **UEFN device development**: `creative_device`, `@editable` properties, lifecycle hooks
- **Safety guidance**: avoid invented APIs; prefer Epic templates and compiler diagnostics

## Recommended architecture

```text
Pi / Codex / VS Code local model
  -> verse-uefn skill
  -> UEFN MCP server
      -> Verse workspace tools
      -> UEFN editor HTTP listener
          -> Unreal Python API
```

The skill tells the agent how to behave. The MCP server gives it structured access to the project and editor. Epic's VS Code Verse extension remains the source of truth for language support and diagnostics.

## Contributing

Issues and PRs welcome at https://github.com/vl4dt/verse-uefn-skill

## License

MIT
