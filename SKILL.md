---
name: verse-uefn
description: Epic Games' Verse programming language and UEFN (Unreal Editor for Fortnite) development toolkit. Covers Verse syntax, types, failable expressions, effects, concurrency, creative_device/NPC patterns, multiplayer, persistence, and @editable properties. Use when writing, reviewing, or debugging Verse code (.verse files), building UEFN devices, NPC behaviors, custom player classes, or any Fortnite Creative 2.0 programming task.
---

# Verse & UEFN Development

Expert guidance for Epic Games' Verse language and Unreal Editor for Fortnite (UEFN). Verse is a statically-typed, multi-paradigm language blending functional, logic, and imperative traditions — designed for the metaverse with immutable-by-default data, first-class failure, explicit effects, and structured concurrency.

## Quick start

Every Verse file starts with `using` imports and defines classes/modules at module scope:

```verse
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
using { /UnrealEngine.com/Temporary/Diagnostics }

my_device := class(creative_device):
    @editable
    Label : string = "My Device"

    OnBegin<override>()<suspends>:void =
        Print("Hello from {Self.Label}!")
```

Place the device in UEFN → Verse compiles → it appears in Content Browser → drag into scene.

## Core language concepts (cheat sheet)

| Concept | Syntax | Notes |
|---------|--------|-------|
| Immutable var | `X:int = 5` | Default; cannot reassign |
| Mutable var | `var X:int = 5` | Requires explicit type + init |
| Type inference | `X := "hello"` | Cannot use with `var` |
| Mutate value | `set X = 10` | Requires `<reads>` + `<writes>` |
| Failable call | `player[Agent]` | Square brackets — may fail |
| Must-succeed call | `Print("hi")` | Parens — must succeed |
| Failure context | `if (Val := Get[]): ...` | Handles both success/fail |
| Suspend fn | `<suspends>` | Async, can Sleep/spawn |
| Decide fn | `<decides>` | May fail |
| Pure fn | `<computes>` | No side effects |
| Transaction | `<transacts>` | reads + writes + allocates |

## Workflows

### Create a custom device
1. In UEFN Verse Explorer → New → "Verse Device" template
2. Class inherits `creative_device` from `/Fortnite.com/Devices`
3. Add `@editable` fields for UE editor exposure
4. Override `OnBegin()` for init, `OnEnd()` for cleanup
5. Save → Verse compiles → device appears in Content Browser

### Create custom NPC behavior
1. In UEFN Verse Explorer → New → "NPC Behavior" template
2. Class inherits `creative_npc_agent` from `/Fortnite.com/NPC`
3. Override movement/behavior methods
4. Assign to NPC device in scene

### Handle player joins/leaves (multiplayer)
- Subscribe to `game_instance.PlayerJoinedEvent` / `PlayerLeftEvent`
- Track players in a `map<int, player>` keyed by player index
- Always validate player references with failable access before use
- See [REFERENCE.md](REFERENCE.md) § Multiplayer Patterns

### Work with persistence
- Mark structs/classes `<persistable>` for save/load
- Use `SaveGame()` / `LoadGame()` on `game_instance`
- Follow Verse Persistence Best Practices (see REFERENCE.md)

## Key patterns
- **Failable expressions**: Use `[]` for operations that may fail; handle in `if`/`for` failure contexts
- **Speculative execution**: Failed `<decides>` calls auto-rollback state changes
- **Structured concurrency**: `sync:`, `race:`, `rush:`, `branch:` for safe parallelism
- **Event-driven**: Subscribe to device events with `.Subscribe()`

## References
- Full language reference → [REFERENCE.md](REFERENCE.md)
- Working code examples → [EXAMPLES.md](EXAMPLES.md)
- Book of Verse (community) → https://verselang.github.io/book/
- Epic Verse API Reference → https://dev.epicgames.com/documentation/fortnite/verse-api
- UEFN Central → https://uefncentral.com

## Best practices checklist
- [ ] Prefer immutable (`:=`) over mutable (`var`/`set`)
- [ ] Use failable `[]` for lookups; handle failures explicitly
- [ ] Mark functions with correct effects (`<computes>`, `<reads>`, `<transacts>`, `<decides>`, `<suspends>`)
- [ ] Validate player references before use in multiplayer
- [ ] Use `<persistable>` only for data that truly needs saving
- [ ] Keep `OnBegin` lightweight; offload heavy work to spawned coroutines
- [ ] Unsubscribe events in `OnEnd` to prevent leaks
