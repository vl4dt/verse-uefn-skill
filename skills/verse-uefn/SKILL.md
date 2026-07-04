---
name: verse-uefn
description: Epic Games Verse and UEFN development workflow for agents. Use when writing, reviewing, or debugging .verse files, creating UEFN devices or NPC behaviors, interpreting Verse diagnostics, or coordinating VS Code, Epic's Verse extension, and the UEFN MCP server.
---

# Verse and UEFN Development

Use this skill for UEFN gameplay code written in Verse. Treat Epic's VS Code Verse extension and UEFN compiler diagnostics as the source of truth. Treat the MCP server as the bridge for project context, safe file operations, editor state, and logs.

## Agent workflow

1. Establish project context before editing.
   - If MCP tools are available, call `uefn_get_project_context` first.
   - Confirm the Verse workspace root matches the open UEFN project.
   - If UEFN is running, call `validate_project_state` before editor-side automation.
2. Read existing project patterns.
   - Use `verse_list_files` and `verse_read_file` to inspect nearby Verse files.
   - Reuse imports, module style, naming, and device patterns already present.
3. Make narrow Verse edits.
   - Prefer `verse_write_file` or `verse_create_device` for files under the detected project root.
   - Do not write outside the UEFN project root.
4. Validate through the real toolchain.
   - Use VS Code/Epic Verse extension diagnostics when visible.
   - Use `verse_get_diagnostics` and UEFN/editor logs for MCP-visible errors.
   - Iterate on diagnostics rather than guessing APIs.

## Python vs Verse

- Verse is runtime gameplay logic: devices, NPC behaviors, player-facing rules, persistence, and simulation.
- Python is UEFN editor automation: placing/configuring actors, asset queries, validation, and batch editor work.
- MCP coordinates both. Use Verse tools for project files and diagnostics; use UEFN editor tools only for live editor state or safe editor automation.

## Minimal device template

Every Verse file starts with `using` imports and defines classes/modules at module scope:

```verse
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
using { /UnrealEngine.com/Temporary/Diagnostics }

agent_safe_device := class(creative_device):
    @editable
    Label : string = "My Device"

    OnBegin<override>()<suspends>:void =
        Print("Hello from {Label}!")
```

Create new devices through UEFN Verse Explorer when working manually, or through `verse_create_device` when an MCP server is connected. After creation, compile/check diagnostics in UEFN or VS Code.

## Core language concepts (cheat sheet)

| Concept | Syntax | Notes |
|---------|--------|-------|
| Immutable var | `X:int = 5` | Default; cannot reassign |
| Mutable var | `var X:int = 5` | Requires explicit type + init |
| Type inference | `X := "hello"` | Cannot use with `var` |
| Mutate value | `set X = 10` | Requires the right effect context |
| Failable call | `player[Agent]` | Square brackets — may fail |
| Must-succeed call | `Print("hi")` | Parens — must succeed |
| Failure context | `if (Val := Get[]): ...` | Handles both success/fail |
| Suspend fn | `<suspends>` | Async, can Sleep/spawn |
| Decide fn | `<decides>` | May fail |
| Pure fn | `<computes>` | No side effects |
| Transaction | `<transacts>` | reads + writes + allocates |

## UEFN workflows

### Create a custom device
1. In UEFN Verse Explorer, create a "Verse Device" template, or use `verse_create_device`.
2. Class inherits `creative_device` from `/Fortnite.com/Devices`
3. Add `@editable` fields for UE editor exposure
4. Override `OnBegin()` for init, `OnEnd()` for cleanup
5. Save, compile, read diagnostics, and place the generated device in UEFN

### Create custom NPC behavior
1. Start from Epic's current NPC Behavior template in Verse Explorer.
2. Keep generated imports and base classes unless diagnostics require changes.
3. Add behavior incrementally and validate after each step.
4. Assign the behavior to the relevant NPC device in UEFN.

### Handle player joins/leaves (multiplayer)
- Prefer device events and APIs already used in the project.
- Always validate agent/player references with failable access before use.
- Avoid inventing global game-instance APIs; verify lifecycle hooks against diagnostics and Epic docs.
- See [REFERENCE.md](REFERENCE.md) § Multiplayer Patterns

### Work with persistence
- Use Epic's current persistable-data patterns from the UEFN docs.
- Keep persistent data schemas small and versionable.
- Validate persistence examples in UEFN before treating them as production code.

## Key patterns
- **Failable expressions**: Use `[]` for operations that may fail; handle in `if`/`for` failure contexts
- **Speculative execution**: Failed `<decides>` calls auto-rollback state changes
- **Structured concurrency**: `sync:`, `race:`, `rush:`, `branch:` for safe parallelism
- **Event-driven**: Subscribe to device events with `.Subscribe()`

## References
- Full language reference → [REFERENCE.md](REFERENCE.md)
- Agent-safe code examples → [EXAMPLES.md](EXAMPLES.md)
- Epic Programming with Verse → https://dev.epicgames.com/documentation/en-us/fortnite/programming-with-verse-in-unreal-editor-for-fortnite
- Book of Verse (community) → https://verselang.github.io/book/
- Epic Verse API Reference → https://dev.epicgames.com/documentation/fortnite/verse-api
- UEFN Central → https://uefncentral.com

## MCP tool map

When available, prefer these tools:

| Need | Tool |
|------|------|
| Confirm project/editor context | `uefn_get_project_context`, `validate_project_state` |
| Inspect Verse files | `verse_list_files`, `verse_read_file` |
| Create/update Verse files | `verse_create_device`, `verse_write_file` |
| Diagnose compile/runtime issues | `verse_get_diagnostics`, `get_editor_log` |
| Safe editor inspection | `get_project_info`, `list_actors`, `list_assets` |

Only use dangerous editor tools such as `execute_python`, asset deletion, actor deletion, or shutdown when the user explicitly asks and the MCP server requires confirmation.

## Best practices checklist
- [ ] Prefer immutable (`:=`) over mutable (`var`/`set`)
- [ ] Use failable `[]` for lookups; handle failures explicitly
- [ ] Mark functions with correct effects (`<computes>`, `<reads>`, `<transacts>`, `<decides>`, `<suspends>`)
- [ ] Validate player references before use in multiplayer
- [ ] Use `<persistable>` only for data that truly needs saving
- [ ] Keep `OnBegin` lightweight; offload heavy work to spawned coroutines
- [ ] Re-run Verse diagnostics after each edit
- [ ] Prefer Epic templates over hand-written base classes for devices and NPC behavior
