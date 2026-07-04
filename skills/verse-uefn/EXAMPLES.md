# Verse and UEFN Agent-Safe Examples

These examples are intentionally conservative. Use them as starting points, then validate with UEFN, VS Code's Epic Verse extension, or MCP `verse_get_diagnostics`.

## Minimal Creative Device

```verse
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
using { /UnrealEngine.com/Temporary/Diagnostics }

hello_device := class(creative_device):

    @editable
    Label : string = "Hello"

    OnBegin<override>()<suspends>:void =
        Print("Device started: {Label}")
```

## Timed Device Loop

```verse
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
using { /UnrealEngine.com/Temporary/Diagnostics }

timed_loop_device := class(creative_device):

    @editable
    TickSeconds : float = 1.0

    @editable
    Enabled : logic = true

    var TickCount : int = 0

    OnBegin<override>()<suspends>:void =
        loop:
            if (Enabled = false):
                break

            set TickCount += 1
            Print("Tick {TickCount}")
            Sleep(TickSeconds)
```

## Editable Device References

Use `@editable` for devices placed and wired in UEFN. Check Epic's current device API reference for the exact event and method names of the device type you are using.

```verse
using { /Fortnite.com/Devices }
using { /UnrealEngine.com/Temporary/Diagnostics }

button_logger_device := class(creative_device):

    @editable
    Button : button_device = button_device{}

    OnBegin<override>()<suspends>:void =
        Button.InteractedWithEvent.Subscribe(OnButtonInteracted)
        Print("Button logger ready")

    OnButtonInteracted(Agent : agent):void =
        Print("Button interacted")
```

## Small State Container

Keep local gameplay state simple. Add effects only when diagnostics show they are required.

```verse
counter_state := class:
    var Count : int = 0

    Increment()<transacts>:void =
        set Count += 1

    Reset()<transacts>:void =
        set Count = 0
```

## MCP-Assisted Edit Loop

Use this sequence when the UEFN MCP server is available:

1. Call `uefn_get_project_context`.
2. Call `verse_list_files` and inspect nearby code with `verse_read_file`.
3. Create a file with `verse_create_device` or update one with `verse_write_file`.
4. Trigger the normal UEFN/VS Code Verse compile flow.
5. Call `verse_get_diagnostics` and fix one compiler/runtime issue at a time.

## Patterns to Avoid

- Do not invent global APIs such as a generic `GetGameContext()` or `SaveGame()` call unless they already exist in the current project or Epic docs.
- Do not use Python-style syntax such as `pass`, `return`-driven early exits, or `f"..."` strings in Verse examples.
- Do not assume direct player mutation methods such as setting location, health, or inventory. Use current Fortnite device APIs and validate with diagnostics.
- Do not write Verse files outside the detected UEFN project root.
