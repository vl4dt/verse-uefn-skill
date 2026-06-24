# Verse Language Reference

Comprehensive reference for Epic Games' Verse programming language as used in UEFN (Unreal Editor for Fortnite). Based on the Book of Verse (verselang.github.io/book/) and Epic's official documentation.

---

## Table of Contents
- [Modules & Imports](#modules--imports)
- [Primitive Types](#primitive-types)
- [Container Types](#container-types)
- [Mutability](#mutability)
- [Functions](#functions)
- [Effect System](#effect-system)
- [Failure System](#failure-system)
- [Control Flow](#control-flow)
- [Structs & Enums](#structs--enums)
- [Classes & Interfaces](#classes--interfaces)
- [Concurrency](#concurrency)
- [UEFN-Specific Patterns](#uefn-specific-patterns)
- [Multiplayer Patterns](#multiplayer-patterns)
- [Persistence](#persistence)

---

## Modules & Imports

Every `.verse` file is a module. Import other modules with `using`:

```verse
using { /Fortnite.com/Devices }
using { /Fortnite.com/NPC }
using { /Verse.org/Simulation }
using { /UnrealEngine.com/Temporary/Diagnostics }
```

- Paths are absolute, starting with `/`
- Module resolution is path-based (no circular dependencies)
- `@package` attribute can control package visibility

## Primitive Types

| Type | Description | Example |
|------|-------------|---------|
| `int` | 64-bit signed integer | `X:int = 42` |
| `float` | 64-bit float (double) | `Y:float = 3.14` |
| `string` | Unicode text | `Name:string = "Verse"` |
| `logic` | Boolean (`true`/`false`) | `IsValid:logic = true` |
| `void` | No return value | Functions returning nothing |

## Container Types

### Optionals (`?T`)
Represent a value that may or may not exist:

```verse
MaybePlayer:?player = false           # Empty optional
MaybePlayer:?player = option{Player}  # Contains value

if (MaybePlayer?):
    Print("Has player!")

# Failable access
ActualPlayer := MaybePlayer?  # Fails if empty
```

### Tuples (`(T1, T2, ...)`)
Fixed-size ordered collections:

```verse
Point:(float, float) = (1.0, 2.0)
X, Y := Point  # Destructure
```

### Arrays (`[]T`)
Dynamic arrays with failable indexing:

```verse
Items:[]string = array{"a", "b", "c"}
First := Items[0]     # Failable — fails if out of bounds
for (Item : Items):
    Print(Item)
```

### Maps (`map<K,V>`)
Key-value stores with failable lookup:

```verse
Scores:map<string, int> = map{}
Scores["Player1"] = 100
if (Score := Scores["Player1"]):
    Print("Score: {Score}")
```

## Mutability

**Immutable by default.** Use `var` for mutable variables and `set` to mutate:

```verse
Constant:int = 42          # Immutable — cannot change
var Mutable:int = 0        # Mutable — requires explicit type + initial value
set Mutable = 10           # Reassign with `set`
set Mutable += 5           # Compound assignment
```

Key rules:
- `var` requires explicit type annotation (no `:=` inference)
- `var` in local scope requires initial value
- `var` fields in classes can omit default (provided at construction)
- Reading `var` requires `<reads>` effect; `set` requires `<writes>`

### Deep vs shallow mutability
- **Structs**: Value types — mutation creates copies (shallow)
- **Classes**: Reference types — mutation affects the shared instance (deep)

## Functions

```verse
# Basic function (pure computation)
Add(A:int, B:int)<computes>:int = A + B

# Function with side effects
UpdateScore(NewScore:int)<transacts>:void =
    set Self.CurrentScore = NewScore

# Failable function
FindPlayer(Index:int)<decides>:player =
    Players[Index]  # May fail if index out of bounds

# Async function
WaitAndPrint()<suspends>:void =
    Sleep(1.0)
    Print("Done!")
```

### Open-world vs closed-world
- **Open-world**: Can call any function (default for methods)
- **Closed-world**: Restricted to a known set of calls (used in `<computes>` contexts)

## Effect System

Effects declare what a function can do beyond computing its return value. They are compiler-enforced contracts.

### Effect families

| Family | Purpose |
|--------|---------|
| Cardinality | Whether/how a function returns |
| Heap | Access to mutable memory |
| Suspension | Whether a function may suspend |
| Divergence | Whether a function may run forever |
| Prediction | Where a function runs (client/server) |

### Effect specifiers

| Specifier | Category | Implies | Description |
|-----------|----------|---------|-------------|
| `<computes>` | Exclusive | — | Pure computation, no side effects |
| `<reads>` | Additive | reads | Can read mutable state |
| `<writes>` | Additive | reads, writes | Can modify mutable state |
| `<allocates>` | Additive | allocates | Can allocate memory |
| `<transacts>` | Exclusive | reads, writes, allocates | Full heap access (common default) |
| `<decides>` | Additive | — | Function may fail |
| `<suspends>` | Additive | — | Function may suspend execution |
| `<converges>` | Exclusive | — | Always returns (no divergence) |

### Effect composition
- A function can have **at most one** exclusive specifier
- Multiple additive specifiers are allowed: `<computes><decides>` is valid
- Effects propagate up the call chain — callers must declare at least what callees declare
- `<computes><transacts>` is an error (two exclusive specifiers)

### Effect rules
- Calling a `<decides>` function requires the caller to also be `<decides>`
- Calling a `<suspends>` function requires the caller to also be `<suspends>`
- `<transacts>` implies `<reads>`, `<writes>`, and `<allocates>`
- Default effect for methods is typically `<transacts>`

## Failure System

Verse uses **failure as first-class control flow** instead of exceptions or boolean checks.

### Failable expressions
Operations that may fail use square brackets `[]`:

```verse
# Array indexing (fails if out of bounds)
Value := Items[Index]

# Map lookup (fails if key missing)
Score := Scores["Player1"]

# Custom failable function
ValidPlayer := FindPlayer[AgentID]
```

### Failure contexts
Failable expressions can only appear in failure contexts:

```verse
# if-else (most common)
if (Value := Items[Index]):
    Print("Got: {Value}")
else:
    Print("Index out of bounds")

# for loop (filters iterations)
for (Item : Inventory, IsWeapon[Item]):
    Equip(Item)

# Failable function body
FindBest(Items:[]item)<decides>:item =
    var Best:?item = false
    for (Item : Items):
        if (IsGood[Item], not Best?, Best.Value < Item.Value):
            set Best = option{Item}
    Best?  # Fails if no good item found
```

### Speculative execution
When a `<decides>` call fails, any state changes it made are **automatically rolled back**:

```verse
Transfer(Amount:int, From:player, To:player)<transacts><decides>:void =
    From.Balance -= Amount   # Rolls back if To's deposit fails
    To.Deposit[Amount]       # May fail (e.g., cap reached)
    From.Balance is restored automatically on failure
```

### The `false?` and `true?` patterns
```verse
false?  # Always fails (useful for early-exit)
true?   # Always succeeds (rarely needed)
```

## Control Flow

### If expressions
If is an expression (produces a value):

```verse
Result := if (Condition):
    "Yes"
else:
    "No"
```

### For loops
For iterates and can filter with failable expressions:

```verse
# Basic iteration
for (Item : Array):
    Process(Item)

# With filtering
for (Item : Array, Item.IsValid[]):
    Process(Item)

# Loop expression (collects results)
Doubled := for (X : Numbers):
    X * 2
```

### Case expressions
Pattern matching on enums and values:

```verse
Result := case(Value):
    1 => "One"
    2 => "Two"
    _ => "Other"  # Default case
```

## Structs & Enums

### Structs (value types)
Immutable by default, copied on assignment:

```verse
item_stats := struct<computes>:
    Damage:float = 0.0
    Defense:float = 0.0
    Weight:float = 1.0

Stats := item_stats{Damage := 25.0, Weight := 2.0}
```

### Enums (fixed named values)
```verse
item_rarity := enum<persistable>:
    common
    uncommon
    rare
    epic
    legendary

Rarity := item_rarity.rare
```

## Classes & Interfaces

### Classes (reference types)
Support inheritance, methods, and mutable fields:

```verse
game_item := class<final><unique><persistable>:
    Name:string
    Rarity:item_rarity = item_rarity.common
    var Health:int = 100

    TakeDamage(Amount:int)<transacts>:void =
        set Health -= Amount

    IsAlive()<decides>:void =
        Health > 0
```

### Class construction
```verse
Item := game_item{Name := "Sword", Rarity := item_rarity.rare}
# Unspecified fields use defaults
BasicItem := game_item{Name := "Stick"}
```

### Inheritance
```verse
weapon := class(game_item):
    AttackType:weapon_type = weapon_type.melee

    <override>
    TakeDamage(Amount:int)<transacts>:void =
        # Custom weapon damage logic
        set Health -= (Amount * 0.8)
```

### Interfaces
Define contracts that classes implement:

```verse
damagable := interface:
    var Health:int

    TakeDamage(Amount:int)<transacts>:void
    IsAlive()<decides>:void

my_unit := class(damagable):
    <override>
    var Health:int = 100

    <override>
    TakeDamage(Amount:int)<transacts>:void =
        set Health -= Amount

    <override>
    IsAlive()<decides>:void =
        Health > 0
```

### Blocks for initialization
```verse
logged_entity := class:
    ID:int

    block:
        Print("Creating entity {ID}")
```

## Concurrency

Verse provides **structured concurrency** — all concurrent operations are scoped and cleaned up automatically.

Requires `<suspends>` effect.

### sync (parallel, wait for all)
```verse
Initialize()<suspends>:void =
    sync:
        LoadAssets()
        SetupNetwork()
        InitUI()
    # All three completed here
```

### race (first to complete wins)
```verse
BestResult := race:
    FastPath()
    ReliablePath()
```

### rush (fire-and-forget within scope)
```verse
Process()<suspends>:void =
    rush:
        BackgroundTask()
    Print("Continuing without waiting")
```

### branch (spawn independent coroutine)
```verse
StartLoop()<suspends>:void =
    branch:
        var Counter:int = 0
        while (true):
            set Counter += 1
            Sleep(1.0)
```

### Sleep
```verse
Wait()<suspends>:void =
    Sleep(2.0)  # Pauses for 2 seconds
    Print("Resumed!")
```

## UEFN-Specific Patterns

### creative_device (custom devices)
The base class for all placeable Verse devices:

```verse
using { /Fortnite.com/Devices }

my_device := class(creative_device):
    @editable
    DisplayName:string = "My Device"

    OnBegin<override>()<suspends>:void =
        # Called when game experience begins
        Print("{DisplayName} started")

    OnEnd<override>():void =
        # Called when game experience ends
        # Warning: coroutines spawned here may never execute
```

### @editable properties
Expose device fields in the UEFN Details panel:

```verse
@editable
MaxPlayers:int = 4

@editable
SpawnPoint:vector = vector{}
```

### Device events
Subscribe to built-in device events:

```verse
OnBegin<override>()<suspends>:void =
    StartButton.InteractedWithEvent.Subscribe(Self.OnButtonPressed)

OnButtonPressed(Agent:agent):void =
    if (Player := player[Agent]):
        Print("{Player} pressed the button!")
```

### NPC behavior
Custom AI for NPCs:

```verse
using { /Fortnite.com/NPC }

my_npc := class(creative_npc_agent):
    OnBegin<override>()<suspends>:void =
        # Custom initialization
        MoveTo(GetRandomLocation(), 5.0)
```

### game_instance access
Access the global game state:

```verse
Game := GetGameContext()<>.GetGameInstance()
Players := Game.GetPlayers()
```

## Multiplayer Patterns

### Player lifecycle
```verse
game_manager := class(creative_device):
    var ActivePlayers:map<int, player> = map{}

    OnBegin<override>()<suspends>:void =
        GetGameContext()<>.GetGameInstance().PlayerJoinedEvent.Subscribe(Self.OnPlayerJoined)
        GetGameContext()<>.GetGameInstance().PlayerLeftEvent.Subscribe(Self.OnPlayerLeft)

    OnPlayerJoined(Agent:agent):void =
        if (Player := player[Agent]):
            ActivePlayers[Player.GetPlayerIndex()] = Player
            Print("Player joined: {Player}")

    OnPlayerLeft(Agent:agent):void =
        if (Player := player[Agent]):
            Index := Player.GetPlayerIndex()
            del ActivePlayers[Index]  # Remove from map
            Print("Player left: {Player}")
```

### Safe player reference handling
```verse
# Always validate before use
if (Player := ActivePlayers[PlayerIndex]):
    Player.SetHealth(100)
else:
    Print("Player not found or already left")

# Never assume a player reference is still valid
# Players can leave between checks
```

### Server vs client prediction
- Use `<predicts>` for functions that run on both client and server
- Client-predicted operations must be deterministic
- Server-authoritative state should use `<transacts>` on the server

## Persistence

### Persistable types
Mark structs/classes with `<persistable>` to save/load:

```verse
player_save_data := struct<persistable>:
    PlayerName:string = ""
    Level:int = 1
    XP:int = 0
    Inventory:[]item_id = array{}
```

### Save/Load
```verse
SaveData():void =
    Data := player_save_data{
        PlayerName := Player.GetName(),
        Level := CurrentLevel,
        XP := CurrentXP
    }
    GetGameContext()<>.GetGameInstance().SaveGame(Data)

LoadData()<decides>:player_save_data =
    GetGameContext()<>.GetGameInstance().LoadGame[player_save_data{}]
```

### Best practices
- Only mark truly persistent data as `<persistable>`
- Provide sensible defaults for all fields
- Handle load failures gracefully (failable `LoadGame`)
- Version your save data format for backward compatibility
- Avoid persisting device references or transient game state
