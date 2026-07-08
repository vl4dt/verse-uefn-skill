# verse-uefn-skill

Pi agent skill for **Epic Games Verse** and **UEFN** (Unreal Editor for Fortnite) development. It is designed to guide agents that work with VS Code, Epic's Verse extension, and the UEFN MCP server.

## Installation

### VS Code Chat / GitHub Copilot

VS Code Chat discovers Agent Skills from project and personal skill folders. The skill folder is the inner `skills/verse-uefn` directory that contains `SKILL.md`.

#### Personal install

Use this when you want the skill available in every VS Code workspace:

```powershell
git clone https://github.com/vl4dt/verse-uefn-skill.git "$env:USERPROFILE\verse-uefn-skill"
New-Item -ItemType Directory -Force "$env:USERPROFILE\.copilot\skills"
New-Item -ItemType Junction `
  -Path "$env:USERPROFILE\.copilot\skills\verse-uefn" `
  -Target "$env:USERPROFILE\verse-uefn-skill\skills\verse-uefn"
```

If you already have this repo cloned locally:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.copilot\skills"
New-Item -ItemType Junction `
  -Path "$env:USERPROFILE\.copilot\skills\verse-uefn" `
  -Target "D:\Lab\StudioTun\skills\verse-uefn\skills\verse-uefn"
```

#### Workspace install

Use this when you want the skill committed or shared with one UEFN workspace:

```powershell
Set-Location D:\Path\To\YourUefnProject
New-Item -ItemType Directory -Force ".github\skills"
Copy-Item `
  -Recurse `
  -Force `
  "D:\Lab\StudioTun\skills\verse-uefn\skills\verse-uefn" `
  ".github\skills\verse-uefn"
```

Restart VS Code, open Chat, type `/`, and verify `verse-uefn` appears. You can also run **Chat: Open Customizations** from the Command Palette and check the Skills tab.

Recommended prompt:

```text
Use the verse-uefn skill. Use the UEFN MCP server to inspect project context before editing Verse files.
```

### Pi

```bash
# From npm (once published)
pi install npm:@vl4dt/verse-uefn-skill

# From git
pi install git:github.com/vl4dt/verse-uefn-skill

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
VS Code Chat / Claude Code / Codex / Pi / local model
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
