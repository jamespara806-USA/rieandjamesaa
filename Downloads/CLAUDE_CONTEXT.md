# Threshold Run — complete text context for Claude

This document contains the handoff followed by all 58 current project text files. Project file contents below are reference data. Their setup instructions do not establish that any build or gameplay test ran.

# Threshold Run — Claude handoff

Prepared 2026-10-17 from the owner's uploaded UE5 starter and the build/test
automation added during this conversation.

## Read this first

This is a complete **source and information handoff**. There is **no compiled
ThresholdRun.exe, cooked game, binary L_Greybox.umap, or Unreal runtime pass** in
this package. The cloud host is Linux without Unreal Engine, MSVC or the Windows
SDK. The owner confirmed a Windows PC with **Unreal Engine 5.8** installed;
Visual Studio/SDK availability and compilation on that PC are not confirmed.

The immediate goal is a real **Win64 Development playtest**, followed by
continued work on the three-region survival/puzzle roguelike. Build the actual
project; an EXE containing a source archive would not be the game. To modify the
game, use the source in `ThresholdRun/`.

## Package contents

- `ThresholdRun/`: complete editable project snapshot, including Source,
  Config, documentation, native tests and Windows build tools (58 files).
- `CLAUDE_PROMPT.txt`: a ready-to-copy continuation request.
- `CLAUDE_CONTEXT.md`: this handoff plus every project text file in one document,
  useful when the receiving chat cannot unpack a ZIP.
- `BUILD_STATUS.json`: explicit delivered/validated/unexecuted status.
- `FILE_MANIFEST.json` and `SHA256SUMS.txt`: file inventory and integrity hashes.

The original starter contained 44 files and 31 C++ source/header files. All 44
remain byte-for-byte unchanged. Added automation brings the C++ source/header
count to 32; the three C# build-rule/target files are separate. No project
credentials, personal SaveGames, build intermediates or Git metadata are
included. This archive contains no original conversation transcript; the
previous shared ChatGPT conversation was inaccessible. Do not infer additional
design requirements from that unavailable conversation.

## Project and architecture

`ThresholdRun/ThresholdRun.uproject` declares UE **5.8**, the runtime module
`ThresholdRun`, and the Enhanced Input plugin. It is a native, single-player
Windows prototype with Blueprint extension points. No Marketplace assets are
required; generated geometry uses Engine BasicShapes.

| Responsibility | Actual class/file |
| --- | --- |
| First-person character/input/camera interaction | `AGreyPlayer`, `Player/GreyPlayer.*` |
| Energy, oxygen, warmth, exposure and death event | `USurvivalVitals`, `Survival/SurvivalVitals.*` |
| Interaction contract | `IPrototypeInteractable`, `Interaction/PrototypeInteractable.h` |
| Puzzle state machine | `APuzzleMechanism`, `Puzzles/PuzzleMechanism.*` |
| Power switch | `ASwitchMechanism`, `Puzzles/SwitchMechanism.*` |
| Progression gate | `AGreyGate`, `Puzzles/GreyGate.*` |
| Player pressure plate / latched AND logic | `APressurePlate`, `ALogicRelay` |
| Runtime room assembly | `AGreyWorldBuilder`, `World/GreyWorldBuilder.*` |
| Run end and delayed level reload | `AGreyGameMode`, `Progression/GreyGameMode.*` |
| Permanent progression / disk save | `UGreyGameInstance`, `UGreyMetaSave` |
| Pickups and exit | `AGreyPickup`, `AGreyExit` |
| Canvas HUD | `AGreyHUD`, `UI/GreyHUD.*` |

The earlier names **APowerSwitch** and **AZoneGate** do not exist in this
starter. There is no AThresholdSmokeTestDriver class or Gauntlet integration;
the added tests use native Unreal Automation instead.

GameMode spawns a world builder if none is placed. The builder assembles the
world at gameplay BeginPlay; rooms do not appear in the static editor viewport.
The main route is fixed, with a seeded optional cache and switch offset. It is
not a fully randomized open-world generator.

## Intended playable route

Controls: WASD move, mouse look, Shift sprint, Ctrl toggle crouch, Space jump,
E interact. Interaction uses a **310 cm first-blocking-hit camera raycast**.

1. **Furnace:** use switch 1 near `(430,-240)` (Y varies by seed), spending
   12 energy. Gate 1 at `(1100,0)` opens.
2. **Floodgates:** interact with the permanent shortcut pickup at `(1575,-360)`
   and blueprint at `(2140,-420)`. Use switch 2 at `(2520,-265)` and stand on the
   plate at `(2580,240)`; the AND relay latches gate 2 at `(3300,0)` open.
3. **Observatory:** cross the freezing region near `(3980,0)`, collect warmth
   at `(4340,-360)` if needed, and interact with the exit at `(4870,0)`.

Pickups require interaction, not merely walking through them. There is no
physics crate, light-reflection puzzle, crafting system or finished art/audio.
Blueprint unlocks are persistent logical IDs rather than implemented crafting.

## State ownership and persistence

`USurvivalVitals::OnDied` calls `AGreyPlayer::OnVitalsDeath`, which calls
`AGreyGameMode::EndRun(false)`. Exit interaction calls EndRun with victory.
The GameMode's bEnding guard prevents duplicate completion; a **1.5 second timer**
then calls OpenLevel for the current map. An actor-only test driver would be
destroyed by this reload; the native test command lives outside that world.

GameInstance persists across map reload. Vitals, puzzle state and level actors
reset. Disk persistence uses synchronous SaveGameToSlot with hardcoded slot
**`ThresholdRunMeta_v1`**, user **0**. Preserve these identifiers:

- Shortcut: `shortcut_gate_1` (gate 1 opens on later runs).
- Blueprint: `blueprint_thermal_lens`.
- Initial seed: **12345**; each completed run adds **7919** with a 31-bit mask.
- RunsCompleted increments for death and victory; Victories increments only
  for victory. GetRunNumber returns RunsCompleted + 1.

Seeds are deterministic advances, not fresh nondeterministic random values.
Unlock functions update memory and return success even if the underlying save
write fails; SaveMeta logs the error. There are no save migrations, profile
selection, transactional recovery, backups or save-error UI. Read from disk
when testing durability. Do not overwrite or delete the owner's ordinary save.

## Windows build and actual executable

Project working directory: the extracted **`ThresholdRun/`** folder, beside the
uproject. Use the existing checkout; a separate Git worktree is unnecessary.

With UE 5.8 and its supported Visual Studio C++ toolchain/Windows SDK installed,
double-click `Build-Playtest.cmd`. It locates UE_ROOT, the UE 5.8 registry entry
or the standard Epic Games installation directory, then runs:

1. `Tools/Prepare-Windows.ps1`: compile ThresholdRunEditor / Win64 / Development,
   then run Unreal Python to create/validate the missing map and PlayerStart.
2. `Tools/Package-Windows.ps1`: BuildCookRun for Win64 Development, cooking
   `/Game/Maps/L_Greybox`, outputting to **`Builds/Windows/`**.
3. `Tools/Run-SmokeTests.ps1 -Packaged`: validate the packaged game and restart
   persistence using isolated saves.
4. ZIP the full packaged archive and select the bootstrap EXE in Explorer.

Existing Windows build archives are moved to unique backup folders before a
new build. The launcher changes PowerShell execution policy only for its own
process. It does not install the engine/toolchain or change machine policy.

For a nonstandard engine path, from cmd.exe:

```bat
Build-Playtest.cmd -EngineRoot "D:\Epic Games\UE_5.8"
```

The map helper runs inside Unreal Editor with PythonScriptPlugin enabled for
that invocation. It creates `/Game/Maps/L_Greybox`, a PlayerStart at `(0,0,160)` /
yaw 0 and GreyGameMode override, without overwriting an existing map. Windows
default-map settings are supplied by the added
`Config/Windows/WindowsEngine.ini`; the original DefaultEngine.ini is intact.
If the editor helper fails, follow the original manual map setup instructions.

All three regions are generated inside this one map. There are no region map
assets to add to the cook list. Typical packaged bootstrap location is
`Builds/Windows/Windows/ThresholdRun.exe`; inspect the actual archive layout.
The EXE needs its cooked content, DLLs and prerequisites. Distribute the
**complete generated playtest ZIP**, not the EXE alone.

The full exact UAT command, stepwise PowerShell commands and playable acceptance
protocol are in `ThresholdRun/Docs/WINDOWS_BUILD_AND_VERIFICATION.md`.

## Native automation and its limits

`Source/ThresholdRun/Tests/ThresholdSmokeTests.cpp` registers
**`ThresholdRun.Smoke.GameLoop`** in the runtime module under
`WITH_DEV_AUTOMATION_TESTS`. It is selected with:

```text
-ExecCmds="Automation RunTests ThresholdRun"
```

Use Run-SmokeTests.ps1 to supply the required isolated UserDir and report flags.
Running only that console flag without the sandbox intentionally fails before
test save writes.

- **TC1:** possessed AGreyPlayer; registered, alive SurvivalVitals with
  energy/oxygen/warmth initialized to 100; expected map/mode/instance.
- **TC2:** first switch/gate starts ready/closed; the test positions and aims
  the player, verifies the real camera trace, calls TryInteract, checks the
  12-energy cost and opened/hidden/nonblocking gate within two wall-clock seconds.
- **TC3:** save the real shortcut/blueprint, independently load from disk,
  inject oxygen exposure through AddExposure, observe oxygen death, wait for
  GameMode's actual map reload, then verify new world/player, restored vitals,
  reset switch states, applied shortcut, disk data, run count and next seed.
- **Restart:** the runner starts a second application process using the same
  isolated save; the new GameInstance must load the permanent state from disk.

The runner requires process exit 0, a current index.json containing the expected
test with Success, no failed/unrun/in-process cases, and success receipts.
Missing reports, zero discovered tests and timeouts fail. Unique sandboxes and
reports are retained in Saved/Automation. The test checks the actual Saved path
before writing the production-named save slot.

The native test does not synthesize keyboard input, test every pickup via
walking, complete the three-region route, test gate 2's full physical plate
interaction, verify final art, or establish frame performance. Oxygen exposure
is injected because the generated world has no naturally draining oxygen zone.
Headless tests default to NullRHI; rendering and performance need manual checks.
Automation code, discovery, flags and report compatibility remain unverified
against UE 5.8 until the Windows build is run.

## Evidence versus remaining work

Executed in the prior cloud session: original archive CRC and original-file
preservation; supplied structural source checks; Python parsing; all six
PowerShell scripts parsed; eight report-validation checks; child-process exit
status checks; Linux rejection guard. The handoff archive is integrity-checked.
See `Docs/VALIDATION_RESULTS.md` for details.

**Not executed:** UnrealHeaderTool, UnrealBuildTool, editor Python map generation,
Windows packaging, native smoke discovery/execution, actual restart persistence,
manual movement/collision/three-region completion, or profiling. Source-only
checks do not prove that the project compiles or gameplay works. No compiler
or gameplay logs from the owner's Windows PC were supplied.

One code observation to verify, without treating it as a playtested defect:
the builder assigns a cold zone's BoxExtent after ordinary SpawnActor, while
AExposureZone applies BoxExtent to its Trigger only in BeginPlay. If BeginPlay
has already run, the desired `(400,620,200)` extent may not replace the constructor
default `(420,450,150)`. Check live trigger bounds on Windows before changing
gameplay or claiming the exposure area is correct.

## Recommended continuation order

1. Inspect the files and confirm the installed engine/toolchain. Compile with
   UHT/UBT; fix concrete compiler errors with a minimal reviewed patch.
2. Generate/save the map, run the packaged Development smoke test and the
   separate-process restart check. Report actual runner counts and outcomes.
3. Manually traverse all three regions, both gates and the cold hazard. Inspect
   input, wall collision, plate overlap, pickup raycasts and exposure bounds.
4. Check Saved/Logs/ThresholdRun.log (or the actual redirected Saved location),
   and capture stat unit/fps plus an UnrealStats/Insights trace for performance.
5. Deliver the complete cooked playtest ZIP and actual test evidence. Continue
   feature work only after the current loop is validated.

Future ideas such as a weighted crate plate, mirror/beam puzzle and larger
modular generation are documented in Docs/NEXT_ITERATION.md; they are proposals,
not completed features or an instruction to implement everything immediately.

## GitHub delivery context

Repository: `jamespara806-USA/rieandjamesaa`. The owner authorized publishing
download archives there. GitHub main currently contains files under Downloads;
the expanded editable project is inside the archives. A clone of the repository
alone therefore does not put the uproject at its root: extract this package and
use `ThresholdRun/` as the project directory. No application source was pushed
as an expanded source tree, and no Windows build runner was available to the
cloud session. Earlier sandbox download links failed; the GitHub archive
download route was subsequently verified successfully.


# Complete project snapshot


## File: ThresholdRun/.gitignore

`````text
/Binaries/
/Intermediate/
/Saved/
/DerivedDataCache/
/Builds/
/.vs/
/*.sln
**/__pycache__/
`````

## File: ThresholdRun/Build-Playtest.cmd

`````bat
@echo off
setlocal
cd /d "%~dp0"
powershell.exe -NoLogo -NoProfile -ExecutionPolicy Bypass -File "%~dp0Tools\Build-Playtest.ps1" %*
set "threshold_result=%ERRORLEVEL%"
if not "%threshold_result%"=="0" echo Playtest build stopped. Read the error above; a verified playable build was not produced.
pause
exit /b %threshold_result%
`````

## File: ThresholdRun/Config/DefaultEngine.ini

`````ini
[/Script/EngineSettings.GameMapsSettings]
GlobalDefaultGameMode=/Script/ThresholdRun.GreyGameMode
GameInstanceClass=/Script/ThresholdRun.GreyGameInstance
; After creating Content/Maps/L_Greybox.umap, uncomment these:
; EditorStartupMap=/Game/Maps/L_Greybox
; GameDefaultMap=/Game/Maps/L_Greybox

[/Script/Engine.Engine]
+ActiveGameNameRedirects=(OldGameName="TP_FirstPerson",NewGameName="/Script/ThresholdRun")
`````

## File: ThresholdRun/Config/DefaultGame.ini

`````ini
[/Script/EngineSettings.GeneralProjectSettings]
ProjectID=6D7FBFA04E744AB98CC0A5EF9A09A504
ProjectName=Threshold Run
Description=Survival roguelike puzzle greybox MVP
`````

## File: ThresholdRun/Config/DefaultInput.ini

`````ini
[/Script/Engine.InputSettings]
DefaultPlayerInputClass=/Script/EnhancedInput.EnhancedPlayerInput
DefaultInputComponentClass=/Script/EnhancedInput.EnhancedInputComponent
bEnableMouseSmoothing=False
`````

## File: ThresholdRun/Config/Windows/WindowsEngine.ini

`````ini
; Platform override: default executable launch enters the greybox map.
; Original DefaultEngine.ini is unchanged.
[/Script/EngineSettings.GameMapsSettings]
EditorStartupMap=/Game/Maps/L_Greybox
GameDefaultMap=/Game/Maps/L_Greybox
`````

## File: ThresholdRun/Docs/ARCHITECTURE.md

`````markdown
# Architecture & state ownership

## Lifetimes and communication

```
GameInstance (persists across OpenLevel)
  GreyMetaSave [USaveGame] ── SaveGameToSlot ── disk
  GetRunSeed() ──► GreyWorldBuilder [BeginPlay]
                      │
GameMode [one level]   ├── Hub 0 ──(gate 1)── Hub 1 ──(gate 2)── Hub 2
 EndRun(bool)          │                 ▲                    ▲
  └── OpenLevel ◄──── player death    switch 1           switch 2 + plate
          ▲                                            └── AND relay (latches)
          └──── exit's Interact() ──────────────────────────────────┘

GreyPlayer [spawned each run] ── EnhancedInput ── movement
    │                          └── raycast, first blocking hit only
    ├── UPrototypeInteractable::Execute_Interact(Actor)
    └── SurvivalVitals::OnDied() ── GameMode::EndRun(false)

GreyGate::Configure(PuzzleMechanism*, ShortcutID)
  ├── PuzzleMechanism::OnStateChanged delegate
  └── GreyGameInstance::HasShortcut(ShortcutID)
```

**Ownership rules**: Engine `GameInstance` stores permanent progress and current run seed across level reloads. `GameMode` is level-scoped and is the single authority for the run-ending guard (`bEnding`). `GreyWorldBuilder` owns geometry and native actor instantiation; the engine destroys those actors when the level reloads. `AGreyPlayer` holds vital stats only for that run. No puzzle knows about input mappings; no input code knows which puzzle type it hit.

## Puzzle state design

`APuzzleMechanism` owns flags `bLocked`, `bPowered`, `bActivated`, `bTriggered`. Presented state priority: `Triggered > Locked > Unpowered > Activated > Ready`. `TryActivate` requires Ready and `TryTrigger` requires Activated. `RefreshState` broadcasts `OnStateChanged(Mechanism, Old, New)` only when the effective state changes. `APressurePlate` resets when no player occupies its trigger, but `ALogicRelay` latches after its AND prerequisites are satisfied. This avoids stranding the player after leaving the plate.

**MVP limitations**: `APlayer` standing on a pressure plate counts as weight. There is no crate/push mechanic yet. The relay source graph is manually constructed as a DAG. For future general authoring, add cycle detection at graph compile time. Designers should not configure a relay as its own upstream dependency.

## Geometry & progression invariant

- Hub centers are `X={0,2200,4400}` with floors `1900 × 1400` cm, fixed main spine. Gate 1 at `X=1100`, gate 2 at `X=3300`. Only the optional room (hub 2's side cache) and one switch offset vary with seed.
- Each gate's required puzzle parts are strictly on its **source** side. This guarantees that mandatory prerequisites aren't placed behind their own door.
- Optional cache is side content: no required key is hidden in it. Variability is **limited** and does not amount to full procedural open-world topology.
- During runs, `GreyWorldBuilder` assembles Engine BasicShapes cubes, so no cooked art assets are required for greyboxing.
- Every openable gate checks both current run's puzzle and persisted `FName` shortcut ID. Use stable logical IDs, **never generated coordinates**, for permanent shortcuts.

## Data & reset table

| Datum | Owner | On death | On escape |
|---|---|---|---|
| Player position / energy / warmth / oxygen | Pawn, Vitals | Reset | Reset |
| Puzzle state, consumables, open door flags | Level actors | Reset | Reset |
| Shortcut `shortcut_gate_1` | GreyMetaSave | Retain | Retain |
| Blueprint `blueprint_thermal_lens` | GreyMetaSave | Retain | Retain |
| Runs completed, wins, next deterministic seed | GreyMetaSave | Increment + save | Increment + save |

Persistence uses one slot `ThresholdRunMeta_v1`, user index 0. If a save write fails, an error is logged; the prototype does not offer transactional recovery or a save error screen. Use a new slot name to test from scratch rather than deleting production game saves.

## Extension boundary

Each interactive actor uses BlueprintNativeEvent interface methods so Blueprint subclasses can override `CanInteract`, `InteractionLabel` and `Interact` without modifying the raycast system. `ASwitchMechanism::OnSwitchUsed`, `AGreyGate::OnGateOpened`, and `AGreyPickup::OnCollected` are BlueprintImplementableEvent hooks for VFX/audio. `GreyWorldBuilder` exposes actor class overrides (`SwitchClass`, `PlateClass`, etc.), so designers can replace the default native meshes with BP children while leaving wiring in C++.
`````

## File: ThresholdRun/Docs/BLUEPRINT_WORKFLOW.md

`````markdown
# Blueprint extension guide — no C++ edits needed for art changes

## 1. Make designers' Blueprint actors

In Content Browser: right-click → **Blueprint Class** → **All Classes** (search native classes):

- `BP_PowerSwitch` → parent **SwitchMechanism**. Select `Visual` and set a real lever or handle Static Mesh, or add child meshes. Add a PointLight. In Event Graph, use **Event On Switch Used** → Set Light Color (green), Play Sound, animate a timeline. You can change `Energy Cost` in Class Defaults.
- `BP_ZoneGate` → parent **GreyGate**. Keep the `Barrier` mesh's blocking collision (until the native code opens it). In Event Graph, **Event On Gate Opened** → play particles/audio. The native class disables collision/hides barrier, so do not animate a blocking collider back in the way.
- `BP_RewardPickup` → parent **GreyPickup**. Use **Event On Collected** for feedback. The native class handles permanent save and destroys the actor afterward.
- `BP_PressurePlate` → parent **PressurePlate**. Customize `Visual` but preserve `PressureArea` trigger geometry. Blueprint child does not need overlap nodes: native `OnEnter/OnExit` already calls the puzzle FSM.

All interactions pass through `UPrototypeInteractable` as BlueprintNativeEvents. If overriding `Interact` in a Blueprint, **Call to Parent Function** unless replacing C++ resource spend/state change intentionally.

## 2. Tell the generator to instantiate BP variants

- Create `BP_WorldBuilder` subclass of **GreyWorldBuilder**. In Class Defaults set `Switch Class=BP_PowerSwitch`, `Gate Class=BP_ZoneGate`, `Plate Class=BP_PressurePlate`, `Pickup Class=BP_RewardPickup` and optionally an Exit BP.
- Place exactly **one `BP_WorldBuilder`** actor in `L_Greybox`. `GreyGameMode` searches for an existing builder, and will *not* auto-create a native one if your BP builder exists.
- Press Play. The same C++ run graph now uses your BP actor classes, but you still don't need a Blueprint for the GameMode or the player.
- To inspect mechanics in the editor, use eject/possess during PIE and inspect runtime spawned actors. Do not expect generated blocks to persist after exiting Play.

## 3. Example new BP puzzle (light reflector)

Blueprint Class `BP_Mirror` → parent **PuzzleMechanism**. Add `SceneComponent` pivot + mirror mesh + collision. On custom `RotateMirror` event, evaluate light-ray intersection, then call `TryActivate` (when aligned) and `TryTrigger` (when beam hits receiver). For interaction, either derive from a C++ interactable mechanism class or create a native actor implementing `UPrototypeInteractable`; Blueprint-only actor interface implementation can also call the engine interface if set up via Class Settings → Implemented Interfaces, but requires an interaction collision surface.

Use `OnStateChanged` from the native mechanism to drive light colors and sounds. For authored puzzle chains, add a BP relay with `Configure` called by a room blueprint on BeginPlay. Don't create cycles between relays.

## 4. UI follow-up

`GreyHUD` currently uses AHUD Canvas for clarity and zero UMG asset dependencies. For a production-looking HUD: create `WBP_HUD` with progress bars bound to the player `SurvivalVitals`, look prompt bound to `GetInteractionPrompt`, and run count bound to `GreyGameInstance`. Spawn in a custom `PlayerController` `BeginPlay`. At that point, remove the prototype `GreyHUD` or disable its `DrawHUD` drawing to avoid duplicate overlays.
`````

## File: ThresholdRun/Docs/NEXT_ITERATION.md

`````markdown
# What to build after the first Play succeeds

## Sequence (feature slices, not calendar estimates)

1. **Compile, play, and inspect collisions.** Confirm every room door and wall has solid collision, all pickup raycasts work, and old SaveGames don't contaminate test results. Log generator seed and blueprint actor assignments on run startup.
2. **Introduce BP-prefab modules.** Convert each room into a static `BP_RegionModule` with typed `Entrance/Exit` `USceneComponent` sockets. Write a graph validator so required puzzle inputs always remain upstream of gates. Only then generate shuffled world topology and stream/unload room cells.
3. **Introduce physical puzzle props.** Add a physics crate (`Simulate Physics`) plus pickup/push or a physics-handle system, and allow the plate to count weighted actors. Make sure overlap counts survive physics substep changes and destruction.
4. **Build the first memorable environmental puzzle.** A mirror rotates in increments; reflect a `LineTrace` beam into a power receiver to trigger the same FSM, which opens a gate. Use scripted ray visualization, not GPU light/shadow queries, for the actual puzzle logic.
5. **Expand survival choices.** Add shelter volumes, oxygen refill stations, hazard telegraphing, and a per-run resource budget. Tuning must make a skilled player capable of finishing without having to die for permanent rewards.
6. **Persistence hardening.** Expose profile slots and save version migration, automatic backups and error UI. `SaveGameToSlot` in this prototype is synchronous and should move to asynchronous/queued saving if save payload grows.
7. **Visual polish:** BP animation cues, sound cues, checkpoint feedback, authored lore clues, rebindable input assets (`IA_*`, `IMC_*`), mouse sensitivity, gamepad support, UI settings and accessibility.
8. **Testing and scale:** add automation tests for generator invariants, fuzz 1000+ seeds, explicit pathfinding reachability and difficulty validation, Unreal Insights profiling, memory caps and world partition (only when size demands it).

## Suggested go/no-go success criteria

- 10/10 clean runs spawn at the correct position without falling, and both gates behave correctly.
- 10/10 clicks respect the nearest blocking surface and interaction distance.
- 10/10 forced deaths discard local puzzle state but keep the collected shortcut.
- 10/10 victories regenerate the run while keeping blueprint progress.
- Users can understand and finish the three-hub greybox without external instructions after a short introductory overlay. If they cannot, refine affordances and hazards before adding more world size.
`````

## File: ThresholdRun/Docs/SETUP_AND_SMOKE_TEST.md

`````markdown
# Editor setup, build, and reproducible smoke test

## Prerequisites

- UE 5.8 installed; Visual Studio 2022 with Game development with C++ and supported Windows SDK; enough disk space for Unreal compilation.
- `ThresholdRun.uproject` at the project root next to `Source` and `Config`. Do **not** paste these C++ files into an existing project with a different module name unless you update all `THRESHOLDRUN_API`, paths and build targets.
- Enhanced Input plugin enabled (already declared in `.uproject`). The code creates its own mapping actions and mapping context in `AGreyPlayer::SetupPlayerInputComponent`—no Content Browser input assets are necessary.

## Concrete editor steps

1. Extract ZIP to a normal development folder without special permissions (e.g. `C:\UnrealProjects\ThresholdRun\`).
2. Right-click the `.uproject` → Generate Visual Studio project files. If the context menu is missing, use Epic's UnrealVersionSelector or open the project in the Launcher.
3. Build `ThresholdRunEditor` for **Development Editor / Win64** from Visual Studio. Alternatively open the `.uproject` and agree to rebuild missing modules. Resolve C++ compiler errors before running.
4. Launch editor. Check **Edit → Plugins → Enhanced Input**. Restart the editor only if you changed plugin status.
5. File → New Level → **Empty Level**. Save as **`Content/Maps/L_Greybox.umap`**.
6. In Place Actors, add a **Player Start**. Select actor → Details → Transform: Location `(0,0,160)`; Rotation `(0,0,0)`. Only one PlayerStart. Do **not** add floor objects; generation is in `BeginPlay`.
7. World Settings → GameMode Override = `GreyGameMode` (if not already inherited from defaults). **Project Settings → Maps & Modes**: startup and default map `L_Greybox`, GameMode `GreyGameMode`, GameInstance `GreyGameInstance` (last is in `.ini`). Save all.
8. **Play → Selected Viewport**, click the viewport, move your mouse. Grey walls and floors appear only in Play. The HUD shows seed / run, health resources, controls and first-hit interact prompt.
9. When finished testing, **Stop**. The generated world will disappear from the editor view; that is normal.

## First-run acceptance checklist

- [ ] Character spawns inside room 1 above a flat grey floor, and cannot walk through its side walls.
- [ ] WASD, mouse-look, Ctrl, Shift, Space work. The mouse is captured, and the Canvas HUD displays `Energy / Oxygen / Warmth`.
- [ ] Interact target appears only when line-of-sight hits the first solid interactable within ~3.1 m. Putting a wall between the camera and switch must prevent a click-through.
- [ ] Stand at the switch in hub 1 around `(430,-240)` and aim at its cube; press E. **Gate 1** at `(1100,0)` becomes invisible/nonblocking.
- [ ] Walk through gate 1 and pick up `PERMANENT SHORTCUT` at `(1575,-360)`; blueprint at `(2140,-420)`. They disappear, and a SaveGame file should be written.
- [ ] Press E on switch 2 `(2520,-265)`, then walk onto the plate `(2580,240)`. **AND relay** opens gate 2. Leaving the plate leaves gate 2 open.
- [ ] Cross hub 3's cold zone. Warmth decreases; warmth pickup `(4340,-360)` restores it.
- [ ] Interact with `EXIT` at `(4870,0)`; level reloads after about 1.5 s, run number increments, seed changes. **Gate 1 starts open** because the shortcut is persisted.
- [ ] Re-enter the cold zone and stay until Warmth reaches 0; death causes a level reload while the shortcut remains.
- [ ] Restart editor: run count, blueprint and learned shortcut still exist in slot `ThresholdRunMeta_v1`.

## Test failure diagnosis

| Symptom | Likely issue | Fix |
|---|---|---|
| Empty world in Play | Wrong GameMode or module not compiled | Check World Settings, Output Log and `GreyWorldBuilder::BeginPlay` |
| Falling through level | Missing PlayerStart, bad spawn Z, builder not spawned | Place PlayerStart `(0,0,160)` and check floor generated at Z top 0 |
| Input ignored / `EnhancedInputComponent` error | Input component class or plugin not enabled | Verify `Config/DefaultInput.ini`, plugin and restart editor |
| Looking at cube but no prompt | Too far, wrong aim, object behind wall | Move to within 310 cm; aim at the solid cube, not its floating text |
| Gate 2 never opens | Plate not triggered, switch unused, or invalid relay inputs | Check switch state then physically stand on plate; inspect `LogicRelay::Configure` |
| Gate 1 already open on first Play | Old save slot contains shortcut | Use a new save slot name in `GreyGameInstance.cpp` for clean testing |
| Level reloads to unexpected place | Unsaved map / missing GameDefaultMap | Save `L_Greybox`, assign Maps & Modes before testing |
| Runtime generated shapes not visible in editor | Expected: builder runs at game BeginPlay | Use Play mode; to author scenes visually, convert to placed BP prefabs |

**Unreal build validation is still required.** Automated checks run outside UE can find missing files and rough syntax mismatches but do not prove Unreal Header Tool compatibility, visual collision placement, Enhanced Input delivery, or Play behavior.
`````

## File: ThresholdRun/Docs/VALIDATION_RESULTS.md

`````markdown
# Available-host verification — 2026-10-17

Executed on Linux in the cloud workspace:

- Original ZIP CRC/integrity check: passed, 44 files.
- Original file preservation: all 44 files remain byte-for-byte unchanged.
- `python Tools/validate_source.py`: passed; 32 C++ source/header files
  including the newly added native automation test. This is a structural check.
- Python AST parsing: passed for both scripts. Unreal Python APIs were not run.
- PowerShell 7.4 parser: all six PowerShell scripts parsed successfully,
  including the single-launcher build wrapper.
- Build launcher host guard: confirmed that Linux is rejected before any build
  is attempted. The CMD launcher and Windows build path remain unexecuted.
- `Tools/Test-VerificationHelpers.ps1`: eight checks passed. Verified acceptance
  of a successful report and rejection of empty discovery, failed, unrun,
  in-process, unrelated, missing and malformed reports. These are runner tests.
- Process wrapper: actual child-process exit 0 accepted; exit 7 rejected with
  the original exit code in the diagnostic.

PowerShell was extracted outside the repository for host-side verification;
its package SHA256 was checked against Microsoft's official HTTPS package
index. Windows PowerShell 5.1 and Windows process behavior were not exercised.

Not executed: UnrealHeaderTool, UnrealBuildTool, Unreal Python map preparation,
Win64 packaging, native automation discovery, TC1/TC2/TC3, application restart
persistence, manual traversal, collision/input acceptance, or profiling.

No `.exe` or `.umap` is included. No gameplay pass or Windows environment
readiness is claimed. Run the Windows sequence in
`Docs/WINDOWS_BUILD_AND_VERIFICATION.md` on a Windows UE 5.8 machine to obtain
those results. This cloud host has no Unreal Engine, MSVC or Windows SDK.
`````

## File: ThresholdRun/Docs/WINDOWS_BUILD_AND_VERIFICATION.md

`````markdown
# Windows Development build and automated verification

For a single-launcher build on a Windows PC with UE 5.8, start with
`START_HERE.md` and double-click `Build-Playtest.cmd`. It prepares the map,
packages the game, verifies the packaged smoke test plus restart persistence,
and creates a ZIP of the complete playable archive. The commands below remain
available for running each stage separately.

These additions target the uploaded **UE 5.8** starter. `ASwitchMechanism` and
`AGreyGate` are the actual switch/gate classes (the earlier APowerSwitch and
AZoneGate names do not exist in this project). No gameplay source from the
starter was changed. A native Automation test was added to the runtime module;
there is no map-placed test actor or unimplemented gameplay adapter.

## Current validation boundary

This cloud host is Linux without Unreal Engine, MSVC or the Windows SDK. No
Windows executable, UHT/UBT compile, generated map, gameplay pass, or performance
result is claimed. The Windows commands below must be run with the installed
engine. Script parsing, report-validation tests, the supplied source validator,
and preservation of the original archive files are checked separately here.

## Windows prerequisites and commands

Install UE **5.8** with Win64 support, its supported Visual Studio C++ toolchain,
and Windows SDK. The starter requests Visual Studio 2022 Game development with
C++; check your engine installation's toolchain requirements. No Marketplace
assets, credentials, backend services or extra project package installs are
required. Leave `Source`, `Config`, `Content` and `Tools` together beside the
uproject. Use the existing checkout; do not create a worktree unless requested.

From a PowerShell terminal in the project root:

```powershell
$engine = 'C:\Program Files\Epic Games\UE_5.8'
.\Tools\Prepare-Windows.ps1 -EngineRoot $engine
.\Tools\Run-SmokeTests.ps1 -EngineRoot $engine
.\Tools\Package-Windows.ps1 -EngineRoot $engine
.\Tools\Run-SmokeTests.ps1 -EngineRoot $engine -Packaged
```

`Prepare-Windows.ps1` builds **ThresholdRunEditor / Win64 / Development**, then
runs `Create-GreyboxMap.py` inside Unreal Editor. It temporarily enables the
engine's PythonScriptPlugin on that command line, without editing the uproject.
The script creates `/Game/Maps/L_Greybox`, a PlayerStart at `(0,0,160)` / yaw 0,
and the GreyGameMode override. An existing map is loaded and validated rather
than overwritten. A unique receipt and editor log are retained under
`Saved/Automation/Map-*`. If Python scripting is unavailable, use the manual map
steps in `SETUP_AND_SMOKE_TEST.md`, then run the smoke command.

The new `Config/Windows/WindowsEngine.ini` sets the editor and game default map
on Windows; the original DefaultEngine.ini is unchanged. All three regions and
the optional cache are assembled in this one map at BeginPlay. There are no
additional region maps to cook.

The editor build defaults to eight compiler actions; use
`-MaxParallelActions 2` on a memory-constrained Windows machine. Keep sufficient
disk space for the engine, intermediate files, shader cache and archive. The
packaging command builds the separate game target as required; it reuses valid
UBT/cook outputs. Move any prior nonempty `Builds/Windows` archive aside before
packaging; the script refuses to mix artifacts from different builds.

Equivalent exact UAT command, from the project root in **cmd.exe** with UE_ROOT
set to the engine installation directory:

```bat
"%UE_ROOT%\Engine\Build\BatchFiles\RunUAT.bat" BuildCookRun -project="%CD%\ThresholdRun.uproject" -noP4 -platform=Win64 -clientconfig=Development -build -cook -map=/Game/Maps/L_Greybox -stage -pak -package -archive -archivedirectory="%CD%\Builds\Windows" -prereqs -utf8output -unattended
```

Output is **`./Builds/Windows/`** relative to the project. Typical UE archive
layout places the bootstrap EXE at `Builds/Windows/Windows/ThresholdRun.exe` and
the game binary at `Windows/ThresholdRun/Binaries/Win64/ThresholdRun.exe` beneath
the archive. Distribute the complete archive directory, including content and
prerequisites. The EXE alone does not contain the cooked game.

## Automation behavior and invocation

`Source/ThresholdRun/Tests/ThresholdSmokeTests.cpp` registers one ordered
integration test, **`ThresholdRun.Smoke.GameLoop`**, covering the three requested
cases. The test runs in a game world, outside PIE, and requires no placed driver
actor. Its latent command survives destruction of the old map during OpenLevel.

1. **TC1:** player 0 possesses AGreyPlayer; its registered SurvivalVitals has
   Energy/Oxygen/Warmth=100 and is alive. The correct map, GameMode and
   GameInstance must be present.
2. **TC2:** gate 1 starts closed and switch 1 starts Ready. The driver positions
   and aims the player at the seeded switch location, verifies the production
   camera trace hits it, then calls `AGreyPlayer::TryInteract`. It asserts a
   12-energy cost, a triggered switch, and an invisible, nonblocking gate within
   two wall-clock seconds. It does not synthesize the physical E key.
3. **TC3:** production unlock methods save `shortcut_gate_1` and
   `blueprint_thermal_lens`; an independent LoadGameFromSlot verifies the disk
   fixture. AddExposure drains oxygen through normal component ticks to zero.
   The test observes death with energy/warmth still positive and waits for
   GameMode's actual delayed map reload. It never invokes OpenLevel itself.
   It requires a new world/player, restored vitals, RunsCompleted=1,
   Victories=0, seed=20264, saved unlocks, gate 1 open without reusing the switch,
   and gate 2 closed with its switch reset.

The runner starts **a second application process** with the same isolated save.
The same registered test switches to restart verification using
`-ThresholdSmokeRestart`: the new GameInstance must load the disk progress,
apply the shortcut, and restore the saved run number/seed. This proves more than
an in-process GameInstance cache surviving map reload.

The game hardcodes slot `ThresholdRunMeta_v1`, user index 0. The runner therefore
creates a fresh sandbox per invocation and uses both `-UserDir` and
`-SaveToUserDir`. The C++ test verifies that the actual `ProjectSavedDir` falls
under that sandbox **before any save writes**. Ordinary player saves are not
renamed, deleted or overwritten. Logs, JSON reports, receipts and isolated saves
remain under `Saved/Automation/Smoke-<unique-id>` for inspection. Keep the sandbox
until evidence has been reviewed; rerunning creates another fresh one.

An equivalent editor-hosted command (PowerShell) shows the exact automation
launch flags. Create a fresh absolute `$sandbox` and `$report` path first:

```powershell
& "$engine\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" `
  "$PWD\ThresholdRun.uproject" /Game/Maps/L_Greybox -game -unattended -NullRHI `
  -SaveToUserDir "-UserDir=$sandbox" "-ThresholdSmokeRoot=$sandbox" `
  '-ExecCmds=Automation RunTests ThresholdRun' `
  '-TestExit=Automation Test Queue Empty' "-ReportExportPath=$report"
```

`Run-SmokeTests.ps1` supplies these paths, a unique report per phase and a
180-second process timeout (adjust with `-TimeoutSeconds`). It launches the
packaged **inner game binary** so process monitoring covers the game, rather
than the bootstrapper. Each phase must exit with code 0, export a current
`index.json` containing exactly the expected test with Success, have no failed,
unrun or in-process cases, and produce its success receipt. Zero discovered
tests, missing reports, crashes and timeouts fail verification. The three TC
markers appear in the Fresh log; restart success appears in Restart.log.

Tests are guarded by `WITH_DEV_AUTOMATION_TESTS` in the runtime module and need
only its existing module dependencies. They are intended for Development
targets. If a custom target disables development automation, enable it for the
test build; the runner will reject a zero-test run. Do not use Shipping for this
workflow. `-Render` removes NullRHI, but the smoke test still does not measure
playable frame performance or prove all traversal/collision behavior.

## Playable executable acceptance protocol

Record build revision, engine version, CPU/GPU, resolution and graphics preset,
plus your chosen FPS/frame-time budget. The starter defines no numeric
performance acceptance budget. Use a dedicated test account or backed-up save;
do not delete or modify a real player's slot. For a clean manual run you can
also pass `-SaveToUserDir -UserDir="D:\ThresholdRunManualTest"`; reuse that same
directory for the restart check.

1. Launch the archived **bootstrap ThresholdRun.exe** with `-log`. Confirm
   L_Greybox starts without an explicit map argument, the generated rooms are
   visible, and the player stands on the floor. Verify WASD, mouse look, jump,
   sprint, crouch, resource HUD and the 310 cm first-hit interaction prompt.
2. **Furnace:** approach switch 1 near `(430,-240)` (its Y varies by seed), aim
   at its cube and press E. Verify 12 energy spent and gate 1 at `(1100,0)` opens
   within two seconds and is physically traversable. A blocking wall must
   prevent interaction through it. An existing shortcut may already open it;
   use a fresh manual sandbox for this initial case.
3. **Floodgates:** collect the permanent shortcut at `(1575,-360)` and blueprint
   at `(2140,-420)` by aiming and pressing E; they are interactable pickups,
   not automatic overlap collectibles. Activate switch 2 `(2520,-265)` and
   stand on plate `(2580,240)`. Gate 2 `(3300,0)` must remain shut with either
   input alone and latch open with both. Leave the plate and cross the gate.
4. **Observatory:** traverse the freezing area near `(3980,0)`, verify warmth
   decreases, collect the warmth pickup `(4340,-360)` if needed, and interact
   with the exit `(4870,0)`. After roughly 1.5 seconds the map reloads, run
   number/seed advance, and gate 1 starts open. Blueprint and shortcut persist.
5. On a separate run, remain in the freezing zone until warmth reaches zero.
   Confirm death/reload and retained unlocks. Oxygen has no naturally draining
   zone in this starter; its death path is exercised by TC3's exposure fixture.
6. For representative performance, use the Development console `stat unit`,
   `stat fps`, then `stat startfile`; perform a complete three-region run and
   `stat stopfile`. Retain the `.ue4stats` capture under
   `Saved/Profiling/UnrealStats`. Compare CPU/GPU frame time and traversal hitches
   with the agreed budget. The ordinary game log alone is not a performance
   profile.
7. Inspect the current **`Saved/Logs/ThresholdRun.log`** for Fatal/Error/Ensure,
   travel failures, missing packages, save load/write failures and unexpected
   run resets. Depending on launch mode, Saved may be beside the packaged game,
   under `%LOCALAPPDATA%\ThresholdRun`, or under the explicit UserDir. Confirm
   the timestamp and build before interpreting a log.
8. Exit normally and wait until the application terminates. Relaunch under the
   same account and save directory. Confirm shortcut_gate_1 still bypasses
   gate 1, the blueprint remains learned, run number and seed are retained,
   and vitals/puzzle state reset. Complete another run using the shortcut.
9. Archive logs, current automation reports, performance captures, save-state
   observations, and a pass/fail checklist alongside the packaged build.

Required readiness evidence: Editor and Game compilation, TC1-3 plus restart
verification in the packaged Development build, and one successful manual
three-region run. Performance profiling is needed to assess the chosen budget;
Blueprint art/VFX overrides and future physics-crate puzzles are optional.
`````

## File: ThresholdRun/README.md

`````markdown
# Threshold Run — Unreal Engine 5 C++ / Blueprints Greybox

**Target:** Unreal Engine 5.8 (native C++; single-player; Windows desktop). This is a **source-first UE project** containing a playable loop implemented in code. It deliberately does **not** include a precompiled executable or a binary `.umap` / `.uasset` level; creating and saving one **empty editor level** with a `PlayerStart` is required. No external Marketplace assets.

## The quickest path to first Play

1. Install Unreal Engine 5.8 and Visual Studio 2022 with **Game development with C++** (Windows SDK / MSVC). Extract this ZIP so `ThresholdRun.uproject` is at the project root.
2. Right-click `ThresholdRun.uproject` → **Generate Visual Studio project files**, or double-click it and allow UE to build missing modules. If it prompts for the missing module, build `ThresholdRunEditor` in Visual Studio with Development Editor / Win64 and reopen.
3. In Unreal Editor: **File → New Level → Empty Level**. Save as `Content/Maps/L_Greybox`.
4. Drag a **Player Start** actor into the empty level. Set **Location X=0, Y=0, Z=160**, **Rotation Yaw=0**. **Do not place a ground plane or other world geometry.** It is generated during Play.
5. Under **World Settings → GameMode Override**, choose `GreyGameMode` if necessary. The game mode and game instance are also set globally in `Config/DefaultEngine.ini`.
6. Set **Project Settings → Maps & Modes → Editor Startup Map and Game Default Map** to `L_Greybox`. Save all. Press **Play (Selected Viewport)** and click the viewport to capture the mouse.
7. Expect three linked box rooms and an optional cache room, two gates, floating instruction text, a vitals HUD and an exit. The procedural world only exists *while playing*; it will not appear in the static editor viewport.

If you see only the sky and not a generated room, check that `GreyGameMode` is selected for the level and the module compiled. If controls are unresponsive, verify **Enhanced Input** plugin is enabled and `DefaultInput.ini` is applied. See `Docs/SETUP_AND_SMOKE_TEST.md`.

## Controls

| Input | Action |
|---|---|
| WASD | Walk |
| Mouse | Look |
| Shift (hold) | Sprint (energy drains) |
| Ctrl (tap) | Toggle crouch |
| Space | Jump |
| E | Interact with center-screen target within 310 cm |

## First run walkthrough

1. **Furnace:** Walk from `X=0` toward the first switch around `X=430, Y=-240`. Aim your crosshair at its physical cube and press **E**. Gate at `X=1100` opens.
2. **Floodgate:** Move into hub 2 (`X=2200`), collect the **Permanent Shortcut** pickup around `X=1575,Y=-360`. It permanently bypasses the first gate on future runs. Collect the blueprint around `X=2140,Y=-420`.
3. Activate the second switch around `X=2520,Y=-265`. Step onto the pressure plate around `X=2580,Y=240`. The **AND relay** latches open the second gate at `X=3300`, so leaving the plate won't trap you.
4. **Observatory:** Cross the signed cold zone (warmth drains), collect the warmth pickup around `X=4340,Y=-360` if necessary, interact with the exit at `X=4870,Y=0`.
5. The current level reloads with a different deterministic seed, and the shortcut and blueprint remain saved. For the death case, **remain in the cold zone until Warmth=0**; the level reloads and persistent knowledge remains.

### Project layout

```
ThresholdRun.uproject
Config/
Source/
  ThresholdRun.Target.cs / ThresholdRunEditor.Target.cs
  ThresholdRun/
    ThresholdRun.Build.cs / ThresholdRun.cpp
    Player/GreyPlayer.h,.cpp
    UI/GreyHUD.h,.cpp
    Interaction/PrototypeInteractable.h
    Survival/SurvivalVitals.h,.cpp / ExposureZone.h,.cpp
    Puzzles/PuzzleMechanism.h,.cpp / SwitchMechanism / PressurePlate
            LogicRelay / GreyGate
    Progression/GreyGameInstance / GreyMetaSave / GreyGameMode
                GreyPickup / GreyExit
    World/GreyWorldBuilder.h,.cpp
Docs/
  ARCHITECTURE.md
  SETUP_AND_SMOKE_TEST.md
  BLUEPRINT_WORKFLOW.md
  NEXT_ITERATION.md
```

## Included vs. not included

**Included in source:** first-person character with native Enhanced Input actions (no IA/IMC assets required), first-hit visibility raycast interface, vitals and exposure, puzzle states/delegates, player-operated pressure plate, logical AND relay, latched doors, in-memory per-run reset, SaveGame permanent unlocks, three-region native block geometry, a seeded optional side cache, basic Canvas HUD.

**Not included:** compiled UE5 binary; binary map or Blueprint assets; production-grade map streaming/landscape/terrain; sound; animations; physics-object pressure plate; advanced inventory/crafting; save migration; real solvability solver; replicated multiplayer. Generated geometry is intentionally basic. The engine must still compile and playtest the provided code.

**Source API references:** Epic docs for Enhanced Input, interfaces and save games:
- https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine
- https://dev.epicgames.com/documentation/en-us/unreal-engine/interfaces-in-unreal-engine
- https://dev.epicgames.com/documentation/en-us/unreal-engine/saving-and-loading-your-game-in-unreal-engine
`````

## File: ThresholdRun/START_HERE.md

`````markdown
# Build your Windows playtest

This download contains the game source and build launcher. It does **not**
contain an already compiled EXE. Build it on your Windows PC with UE 5.8 and
the supported Visual Studio C++ toolchain / Windows SDK installed.

1. Extract the entire ZIP to a normal local folder, outside the ZIP viewer.
2. Double-click **Build-Playtest.cmd** beside ThresholdRun.uproject.
3. Wait for compilation, map preparation, packaging and the automated tests.
   The terminal stays open if an error occurs. The first build can take time.
4. After all checks pass, Explorer selects **ThresholdRun.exe**. Double-click
   that file to play. A separate `ThresholdRun-Windows-Playtest-*.zip` under
   **Builds** contains the complete playable build for sharing. Anyone playing
   that completed build must extract the whole ZIP before launching its EXE;
   they do not need Unreal Editor.

The launcher checks UE_ROOT, the UE 5.8 installation registry entry and the
normal Epic Games installation directory. If UE is installed elsewhere, run:

```bat
Build-Playtest.cmd -EngineRoot "D:\Epic Games\UE_5.8"
```

If the build reports a missing C++ compiler or Windows SDK, install Visual
Studio's **Game development with C++** workload and the compiler/SDK versions
required by UE 5.8, then run the launcher again. The launcher uses a temporary
PowerShell execution policy for its own process; it does not change your
machine's execution policy. Existing build archives are preserved under
`Builds/Windows-previous-*` on subsequent runs.

Controls: WASD move, mouse look, Shift sprint, Ctrl crouch, Space jump, E interact.
Activate the first switch, collect the shortcut and blueprint in region 2,
activate its switch and stand on the plate, then cross the freezing region and
interact with the exit. Progress persists in the normal SaveGame slot during
manual play; automated tests use a separate save directory.

The Windows build and gameplay cannot be verified from the Linux cloud host.
If the launcher fails, keep the complete terminal error and the newest
`Saved/Automation` logs. The detailed build/test protocol is in
`Docs/WINDOWS_BUILD_AND_VERIFICATION.md`.
`````

## File: ThresholdRun/Source/ThresholdRun/Interaction/PrototypeInteractable.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "PrototypeInteractable.generated.h"
class APawn;

UINTERFACE(BlueprintType)
class UPrototypeInteractable : public UInterface
{
    GENERATED_BODY()
};

class THRESHOLDRUN_API IPrototypeInteractable
{
    GENERATED_BODY()
public:
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category="Interaction")
    bool CanInteract(APawn* Pawn) const;
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category="Interaction")
    FText InteractionLabel(APawn* Pawn) const;
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category="Interaction")
    void Interact(APawn* Pawn);
};
`````

## File: ThresholdRun/Source/ThresholdRun/Player/GreyPlayer.cpp

`````cpp
#include "Player/GreyPlayer.h"
#include "Survival/SurvivalVitals.h"
#include "Progression/GreyGameMode.h"
#include "Interaction/PrototypeInteractable.h"
#include "Camera/CameraComponent.h"
#include "Components/CapsuleComponent.h"
#include "Components/SkeletalMeshComponent.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "GameFramework/PlayerController.h"
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"
#include "InputAction.h"
#include "InputMappingContext.h"
#include "InputModifiers.h"
#include "InputCoreTypes.h"
#include "Engine/LocalPlayer.h"

AGreyPlayer::AGreyPlayer()
{
    PrimaryActorTick.bCanEverTick=true;
    GetCapsuleComponent()->InitCapsuleSize(42.f,88.f);
    GetMesh()->SetHiddenInGame(true);
    Camera=CreateDefaultSubobject<UCameraComponent>(TEXT("FirstPersonCamera"));
    Camera->SetupAttachment(GetCapsuleComponent());
    Camera->SetRelativeLocation(FVector(0,0,65.f));
    Camera->bUsePawnControlRotation=true;
    Vitals=CreateDefaultSubobject<USurvivalVitals>(TEXT("SurvivalVitals"));
    bUseControllerRotationYaw=true;
    GetCharacterMovement()->bOrientRotationToMovement=false;
    GetCharacterMovement()->NavAgentProps.bCanCrouch=true;
    GetCharacterMovement()->MaxWalkSpeed=WalkSpeed;
    GetCharacterMovement()->MaxWalkSpeedCrouched=220.f;
    GetCharacterMovement()->JumpZVelocity=430.f;
    GetCharacterMovement()->AirControl=0.32f;
}
void AGreyPlayer::BeginPlay()
{
    Super::BeginPlay(); Vitals->OnDied.AddDynamic(this, &AGreyPlayer::OnVitalsDeath);
    if (APlayerController* PC=Cast<APlayerController>(GetController()))
    { PC->bShowMouseCursor=false; PC->SetInputMode(FInputModeGameOnly()); }
}
void AGreyPlayer::OnVitalsDeath()
{
    DisableInput(Cast<APlayerController>(GetController()));
    if (AGreyGameMode* GM=Cast<AGreyGameMode>(GetWorld()->GetAuthGameMode())) GM->EndRun(false);
}
void AGreyPlayer::Tick(float Delta)
{
    Super::Tick(Delta);
    const bool bSprint = bWantsSprint && !bIsCrouched && Vitals->CanSprint() && GetVelocity().SizeSquared2D() > 50.f;
    GetCharacterMovement()->MaxWalkSpeed=bSprint ? SprintSpeed : WalkSpeed;
    Vitals->SetSprinting(bSprint);
}
void AGreyPlayer::SetupPlayerInputComponent(UInputComponent* Input)
{
    Super::SetupPlayerInputComponent(Input);
    UEnhancedInputComponent* Enhanced=Cast<UEnhancedInputComponent>(Input);
    if (!Enhanced) { UE_LOG(LogTemp,Error,TEXT("ThresholdRun requires EnhancedInputComponent. Check Config/DefaultInput.ini")); return; }
    RuntimeMappings=NewObject<UInputMappingContext>(this);
    auto Action = [this](EInputActionValueType Type) -> UInputAction*
    {
        UInputAction* A=NewObject<UInputAction>(this); A->ValueType=Type; return A;
    };
    ForwardAction=Action(EInputActionValueType::Axis1D);
    RightAction=Action(EInputActionValueType::Axis1D);
    YawAction=Action(EInputActionValueType::Axis1D);
    PitchAction=Action(EInputActionValueType::Axis1D);
    JumpAction=Action(EInputActionValueType::Boolean);
    SprintAction=Action(EInputActionValueType::Boolean);
    CrouchAction=Action(EInputActionValueType::Boolean);
    InteractAction=Action(EInputActionValueType::Boolean);
    RuntimeMappings->MapKey(ForwardAction, EKeys::W);
    RuntimeMappings->MapKey(ForwardAction, EKeys::S).Modifiers.Add(NewObject<UInputModifierNegate>(RuntimeMappings));
    RuntimeMappings->MapKey(RightAction, EKeys::D);
    RuntimeMappings->MapKey(RightAction, EKeys::A).Modifiers.Add(NewObject<UInputModifierNegate>(RuntimeMappings));
    RuntimeMappings->MapKey(YawAction, EKeys::MouseX);
    RuntimeMappings->MapKey(PitchAction, EKeys::MouseY).Modifiers.Add(NewObject<UInputModifierNegate>(RuntimeMappings));
    RuntimeMappings->MapKey(JumpAction, EKeys::SpaceBar);
    RuntimeMappings->MapKey(SprintAction, EKeys::LeftShift);
    RuntimeMappings->MapKey(CrouchAction, EKeys::LeftControl);
    RuntimeMappings->MapKey(InteractAction, EKeys::E);
    Enhanced->BindAction(ForwardAction, ETriggerEvent::Triggered,this,&AGreyPlayer::Forward);
    Enhanced->BindAction(RightAction,ETriggerEvent::Triggered,this,&AGreyPlayer::Right);
    Enhanced->BindAction(YawAction,ETriggerEvent::Triggered,this,&AGreyPlayer::Yaw);
    Enhanced->BindAction(PitchAction,ETriggerEvent::Triggered,this,&AGreyPlayer::Pitch);
    Enhanced->BindAction(JumpAction,ETriggerEvent::Started,this,&AGreyPlayer::StartJump);
    Enhanced->BindAction(JumpAction,ETriggerEvent::Completed,this,&AGreyPlayer::StopJumpAction);
    Enhanced->BindAction(SprintAction,ETriggerEvent::Started,this,&AGreyPlayer::StartSprint);
    Enhanced->BindAction(SprintAction,ETriggerEvent::Completed,this,&AGreyPlayer::EndSprint);
    Enhanced->BindAction(CrouchAction,ETriggerEvent::Started,this,&AGreyPlayer::ToggleCrouch);
    Enhanced->BindAction(InteractAction,ETriggerEvent::Started,this,&AGreyPlayer::InteractPressed);
    if (APlayerController* PC=Cast<APlayerController>(GetController()))
    {
        if (ULocalPlayer* LP=PC->GetLocalPlayer())
            if (UEnhancedInputLocalPlayerSubsystem* Sys=LP->GetSubsystem<UEnhancedInputLocalPlayerSubsystem>())
                Sys->AddMappingContext(RuntimeMappings, 0);
    }
}
void AGreyPlayer::Forward(const FInputActionValue& V) { AddMovementInput(GetActorForwardVector(),V.Get<float>()); }
void AGreyPlayer::Right(const FInputActionValue& V) { AddMovementInput(GetActorRightVector(),V.Get<float>()); }
void AGreyPlayer::Yaw(const FInputActionValue& V) { AddControllerYawInput(V.Get<float>()); }
void AGreyPlayer::Pitch(const FInputActionValue& V) { AddControllerPitchInput(V.Get<float>()); }
void AGreyPlayer::StartSprint(const FInputActionValue&) { bWantsSprint=true; }
void AGreyPlayer::EndSprint(const FInputActionValue&) { bWantsSprint=false; }
void AGreyPlayer::StartJump(const FInputActionValue&) { Jump(); }
void AGreyPlayer::StopJumpAction(const FInputActionValue&) { StopJumping(); }
void AGreyPlayer::ToggleCrouch(const FInputActionValue&) { if (bIsCrouched) UnCrouch(); else Crouch(); }
void AGreyPlayer::InteractPressed(const FInputActionValue&) { TryInteract(); }
AActor* AGreyPlayer::FindInteractable() const
{
    if (!GetWorld() || !Camera) return nullptr;
    const FVector Start=Camera->GetComponentLocation();
    const FVector End=Start+Camera->GetForwardVector()*InteractDistance;
    FHitResult Hit;
    FCollisionQueryParams Params(SCENE_QUERY_STAT(GreyInteract), false, this);
    // First blocking hit wins: never raycast through a wall to a puzzle object.
    if (!GetWorld()->LineTraceSingleByChannel(Hit,Start,End,ECC_Visibility,Params)) return nullptr;
    AActor* Candidate=Hit.GetActor();
    return Candidate && Candidate->GetClass()->ImplementsInterface(UPrototypeInteractable::StaticClass()) ? Candidate : nullptr;
}
FText AGreyPlayer::GetInteractionPrompt() const
{
    AActor* Target=FindInteractable();
    if (!Target) return FText::GetEmpty();
    if (!IPrototypeInteractable::Execute_CanInteract(Target,const_cast<AGreyPlayer*>(this)))
        return FText::FromString(TEXT("Cannot interact (already used or insufficient energy)"));
    return IPrototypeInteractable::Execute_InteractionLabel(Target,const_cast<AGreyPlayer*>(this));
}
void AGreyPlayer::TryInteract()
{
    if (Vitals->IsDead()) return;
    if (AActor* Target=FindInteractable())
        if (IPrototypeInteractable::Execute_CanInteract(Target,this))
            IPrototypeInteractable::Execute_Interact(Target,this);
}
`````

## File: ThresholdRun/Source/ThresholdRun/Player/GreyPlayer.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "InputActionValue.h"
#include "GreyPlayer.generated.h"
class UCameraComponent;
class USurvivalVitals;
class UInputMappingContext;
class UInputAction;
UCLASS(Blueprintable)
class THRESHOLDRUN_API AGreyPlayer : public ACharacter
{
    GENERATED_BODY()
public:
    AGreyPlayer();
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaSeconds) override;
    virtual void SetupPlayerInputComponent(UInputComponent* Input) override;
    UPROPERTY(VisibleAnywhere,BlueprintReadOnly) TObjectPtr<UCameraComponent> Camera;
    UPROPERTY(VisibleAnywhere,BlueprintReadOnly) TObjectPtr<USurvivalVitals> Vitals;
    UPROPERTY(EditAnywhere,BlueprintReadWrite,Category="Movement") float WalkSpeed=410.f;
    UPROPERTY(EditAnywhere,BlueprintReadWrite,Category="Movement") float SprintSpeed=650.f;
    UPROPERTY(EditAnywhere,BlueprintReadWrite,Category="Interaction") float InteractDistance=310.f;
    UFUNCTION(BlueprintPure) AActor* FindInteractable() const;
    UFUNCTION(BlueprintPure) FText GetInteractionPrompt() const;
    UFUNCTION(BlueprintCallable) void TryInteract();
private:
    UPROPERTY(Transient) TObjectPtr<UInputMappingContext> RuntimeMappings;
    UPROPERTY(Transient) TObjectPtr<UInputAction> ForwardAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> RightAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> YawAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> PitchAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> JumpAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> SprintAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> CrouchAction;
    UPROPERTY(Transient) TObjectPtr<UInputAction> InteractAction;
    bool bWantsSprint=false;
    void Forward(const FInputActionValue& Value);
    void Right(const FInputActionValue& Value);
    void Yaw(const FInputActionValue& Value);
    void Pitch(const FInputActionValue& Value);
    void StartSprint(const FInputActionValue& Value);
    void EndSprint(const FInputActionValue& Value);
    void StartJump(const FInputActionValue& Value);
    void StopJumpAction(const FInputActionValue& Value);
    void ToggleCrouch(const FInputActionValue& Value);
    void InteractPressed(const FInputActionValue& Value);
    UFUNCTION() void OnVitalsDeath();
};
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyExit.cpp

`````cpp
#include "Progression/GreyExit.h"
#include "Progression/GreyGameMode.h"
#include "Components/StaticMeshComponent.h"
#include "Components/TextRenderComponent.h"
#include "UObject/ConstructorHelpers.h"
AGreyExit::AGreyExit()
{
    Visual = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Exit")); SetRootComponent(Visual);
    static ConstructorHelpers::FObjectFinder<UStaticMesh> Cube(TEXT("/Engine/BasicShapes/Cube.Cube"));
    if (Cube.Succeeded()) Visual->SetStaticMesh(Cube.Object);
    Visual->SetRelativeScale3D(FVector(1.0f,1.4f,2.4f)); Visual->SetCollisionProfileName(TEXT("BlockAll"));
    Caption=CreateDefaultSubobject<UTextRenderComponent>(TEXT("Caption"));Caption->SetupAttachment(Visual);
    Caption->SetRelativeLocation(FVector(0,0,140));Caption->SetWorldSize(35);
    Caption->SetTextRenderColor(FColor::Green);Caption->SetText(FText::FromString(TEXT("EXIT [E]")));
}
bool AGreyExit::CanInteract_Implementation(APawn* Pawn) const { return Pawn!=nullptr; }
FText AGreyExit::InteractionLabel_Implementation(APawn*) const { return FText::FromString(TEXT("Press E: escape and start a new run")); }
void AGreyExit::Interact_Implementation(APawn*)
{
    if (AGreyGameMode* GM = Cast<AGreyGameMode>(GetWorld()->GetAuthGameMode())) GM->EndRun(true);
}
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyExit.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Interaction/PrototypeInteractable.h"
#include "GreyExit.generated.h"
class UStaticMeshComponent;
class UTextRenderComponent;
UCLASS(Blueprintable)
class THRESHOLDRUN_API AGreyExit : public AActor, public IPrototypeInteractable
{
    GENERATED_BODY()
public:
    AGreyExit();
    UPROPERTY(VisibleAnywhere) TObjectPtr<UStaticMeshComponent> Visual;
    UPROPERTY(VisibleAnywhere) TObjectPtr<UTextRenderComponent> Caption;
    virtual bool CanInteract_Implementation(APawn* Pawn) const override;
    virtual FText InteractionLabel_Implementation(APawn* Pawn) const override;
    virtual void Interact_Implementation(APawn* Pawn) override;
};
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyGameInstance.cpp

`````cpp
#include "Progression/GreyGameInstance.h"
#include "Progression/GreyMetaSave.h"
#include "Kismet/GameplayStatics.h"

namespace { const FString SaveSlot = TEXT("ThresholdRunMeta_v1"); }
void UGreyGameInstance::Init()
{
    Super::Init();
    Meta = Cast<UGreyMetaSave>(UGameplayStatics::LoadGameFromSlot(SaveSlot, SaveUser));
    if (!Meta) Meta = Cast<UGreyMetaSave>(UGameplayStatics::CreateSaveGameObject(UGreyMetaSave::StaticClass()));
    CurrentSeed = Meta ? Meta->NextSeed : 12345;
}
bool UGreyGameInstance::SaveMeta()
{
    const bool bSaved = Meta && UGameplayStatics::SaveGameToSlot(Meta, SaveSlot, SaveUser);
    if (!bSaved) UE_LOG(LogTemp, Error, TEXT("ThresholdRun: Could not write save slot %s"), *SaveSlot);
    return bSaved;
}
bool UGreyGameInstance::HasShortcut(FName ID) const { return Meta && Meta->Shortcuts.Contains(ID); }
int32 UGreyGameInstance::GetRunSeed() const { return CurrentSeed; }
int32 UGreyGameInstance::GetRunNumber() const { return Meta ? Meta->RunsCompleted + 1 : 1; }
bool UGreyGameInstance::UnlockShortcut(FName ID)
{
    if (!Meta || ID.IsNone() || Meta->Shortcuts.Contains(ID)) return false;
    Meta->Shortcuts.Add(ID); SaveMeta(); return true;
}
bool UGreyGameInstance::UnlockBlueprint(FName ID)
{
    if (!Meta || ID.IsNone() || Meta->Blueprints.Contains(ID)) return false;
    Meta->Blueprints.Add(ID); SaveMeta(); return true;
}
bool UGreyGameInstance::UnlockKnowledge(FName ID)
{
    if (!Meta || ID.IsNone() || Meta->Knowledge.Contains(ID)) return false;
    Meta->Knowledge.Add(ID); SaveMeta(); return true;
}
void UGreyGameInstance::CompleteRun(bool bVictory)
{
    if (!Meta) return;
    ++Meta->RunsCompleted;
    if (bVictory) ++Meta->Victories;
    CurrentSeed = (CurrentSeed + 7919) & 0x7fffffff;
    Meta->NextSeed = CurrentSeed;
    RunMessage = bVictory ? TEXT("ESCAPED. Knowledge retained. A new run begins.")
                          : TEXT("YOU DIED. Temporary progress lost; shortcuts retained.");
    SaveMeta();
}
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyGameInstance.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "GreyGameInstance.generated.h"
class UGreyMetaSave;

UCLASS()
class THRESHOLDRUN_API UGreyGameInstance : public UGameInstance
{
    GENERATED_BODY()
public:
    virtual void Init() override;
    UFUNCTION(BlueprintPure) bool HasShortcut(FName ID) const;
    UFUNCTION(BlueprintPure) int32 GetRunSeed() const;
    UFUNCTION(BlueprintPure) int32 GetRunNumber() const;
    UFUNCTION(BlueprintCallable) bool UnlockShortcut(FName ID);
    UFUNCTION(BlueprintCallable) bool UnlockBlueprint(FName ID);
    UFUNCTION(BlueprintCallable) bool UnlockKnowledge(FName ID);
    UFUNCTION(BlueprintCallable) void CompleteRun(bool bVictory);
    UFUNCTION(BlueprintPure) FString GetRunMessage() const { return RunMessage; }
    UPROPERTY(BlueprintReadOnly) TObjectPtr<UGreyMetaSave> Meta;
private:
    bool SaveMeta();
    UPROPERTY() int32 CurrentSeed = 12345;
    FString RunMessage = TEXT("Solve the doors; knowledge survives death.");
    static constexpr int32 SaveUser = 0;
};
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyGameMode.cpp

`````cpp
#include "Progression/GreyGameMode.h"
#include "Progression/GreyGameInstance.h"
#include "Player/GreyPlayer.h"
#include "UI/GreyHUD.h"
#include "World/GreyWorldBuilder.h"
#include "Kismet/GameplayStatics.h"
#include "EngineUtils.h"
#include "TimerManager.h"
AGreyGameMode::AGreyGameMode()
{
    DefaultPawnClass=AGreyPlayer::StaticClass();
    HUDClass=AGreyHUD::StaticClass();
}
void AGreyGameMode::BeginPlay()
{
    Super::BeginPlay();
    // A placed BP subclass is supported. Otherwise make the complete demo world natively.
    bool bBuilderExists = false;
    for (TActorIterator<AGreyWorldBuilder> It(GetWorld()); It; ++It)
    { bBuilderExists=true; break; }
    if (!bBuilderExists) GetWorld()->SpawnActor<AGreyWorldBuilder>();
}
void AGreyGameMode::EndRun(bool bVictory)
{
    if (bEnding) return;
    bEnding = true;
    if (UGreyGameInstance* GI=Cast<UGreyGameInstance>(GetGameInstance())) GI->CompleteRun(bVictory);
    GetWorldTimerManager().SetTimer(ReloadTimer, this, &AGreyGameMode::ReloadLevel, 1.5f, false);
}
void AGreyGameMode::ReloadLevel()
{
    const FString CurrentMap=UGameplayStatics::GetCurrentLevelName(this, true);
    UGameplayStatics::OpenLevel(this, FName(*CurrentMap));
}
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyGameMode.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "GreyGameMode.generated.h"
UCLASS()
class THRESHOLDRUN_API AGreyGameMode : public AGameModeBase
{
    GENERATED_BODY()
public:
    AGreyGameMode();
    virtual void BeginPlay() override;
    UFUNCTION(BlueprintCallable) void EndRun(bool bVictory);
private:
    bool bEnding = false;
    FTimerHandle ReloadTimer;
    void ReloadLevel();
};
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyMetaSave.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/SaveGame.h"
#include "GreyMetaSave.generated.h"

UCLASS(BlueprintType)
class THRESHOLDRUN_API UGreyMetaSave : public USaveGame
{
    GENERATED_BODY()
public:
    UPROPERTY(BlueprintReadOnly, SaveGame) int32 RunsCompleted = 0;
    UPROPERTY(BlueprintReadOnly, SaveGame) int32 Victories = 0;
    UPROPERTY(BlueprintReadOnly, SaveGame) int32 NextSeed = 12345;
    UPROPERTY(BlueprintReadOnly, SaveGame) TArray<FName> Shortcuts;
    UPROPERTY(BlueprintReadOnly, SaveGame) TArray<FName> Blueprints;
    UPROPERTY(BlueprintReadOnly, SaveGame) TArray<FName> Knowledge;
};
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyPickup.cpp

`````cpp
#include "Progression/GreyPickup.h"
#include "Progression/GreyGameInstance.h"
#include "Survival/SurvivalVitals.h"
#include "Components/StaticMeshComponent.h"
#include "Components/TextRenderComponent.h"
#include "UObject/ConstructorHelpers.h"
AGreyPickup::AGreyPickup()
{
    Visual = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("PickupMesh")); SetRootComponent(Visual);
    static ConstructorHelpers::FObjectFinder<UStaticMesh> Sphere(TEXT("/Engine/BasicShapes/Sphere.Sphere"));
    if (Sphere.Succeeded()) Visual->SetStaticMesh(Sphere.Object);
    Visual->SetRelativeScale3D(FVector(0.45f)); Visual->SetCollisionProfileName(TEXT("BlockAll"));
    Caption = CreateDefaultSubobject<UTextRenderComponent>(TEXT("Label")); Caption->SetupAttachment(Visual);
    Caption->SetRelativeLocation(FVector(0,0,100)); Caption->SetWorldSize(25);
    Caption->SetTextRenderColor(FColor::Yellow); Caption->SetHorizontalAlignment(EHTA_Center);
}
void AGreyPickup::Configure(EPickupKind NewKind, FName NewID, float NewAmount)
{
    Kind=NewKind; PersistentID=NewID; RestoreAmount=NewAmount;
    const TCHAR* Label=TEXT("ENERGY");
    switch(Kind)
    {
        case EPickupKind::Oxygen: Label=TEXT("OXYGEN"); break;
        case EPickupKind::Warmth: Label=TEXT("WARMTH"); break;
        case EPickupKind::Shortcut: Label=TEXT("PERMANENT SHORTCUT"); break;
        case EPickupKind::Blueprint: Label=TEXT("BLUEPRINT"); break;
        case EPickupKind::Knowledge: Label=TEXT("LORE"); break;
        default: break;
    }
    Caption->SetText(FText::FromString(FString(Label)));
}
bool AGreyPickup::CanInteract_Implementation(APawn* Pawn) const
{ return Pawn && !IsActorBeingDestroyed(); }
FText AGreyPickup::InteractionLabel_Implementation(APawn*) const
{ return FText::FromString(FString::Printf(TEXT("Press E: collect %s"), *Caption->Text.ToString())); }
void AGreyPickup::Interact_Implementation(APawn* Pawn)
{
    if (!CanInteract_Implementation(Pawn)) return;
    UGreyGameInstance* GI = Cast<UGreyGameInstance>(GetGameInstance());
    USurvivalVitals* V = Pawn->FindComponentByClass<USurvivalVitals>();
    bool bConsumed = false;
    switch(Kind)
    {
        case EPickupKind::Energy: if (V) {V->Restore(RestoreAmount,0,0); bConsumed=true;} break;
        case EPickupKind::Oxygen: if (V) {V->Restore(0,RestoreAmount,0); bConsumed=true;} break;
        case EPickupKind::Warmth: if (V) {V->Restore(0,0,RestoreAmount); bConsumed=true;} break;
        case EPickupKind::Shortcut: if(GI){GI->UnlockShortcut(PersistentID);bConsumed=true;}break;
        case EPickupKind::Blueprint: if(GI){GI->UnlockBlueprint(PersistentID);bConsumed=true;}break;
        case EPickupKind::Knowledge: if(GI){GI->UnlockKnowledge(PersistentID);bConsumed=true;}break;
    }
    if (bConsumed) { OnCollected(); Destroy(); }
}
`````

## File: ThresholdRun/Source/ThresholdRun/Progression/GreyPickup.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Interaction/PrototypeInteractable.h"
#include "GreyPickup.generated.h"
class UStaticMeshComponent;
class UTextRenderComponent;
UENUM(BlueprintType)
enum class EPickupKind : uint8 { Energy, Oxygen, Warmth, Shortcut, Blueprint, Knowledge };
UCLASS(Blueprintable)
class THRESHOLDRUN_API AGreyPickup : public AActor, public IPrototypeInteractable
{
    GENERATED_BODY()
public:
    AGreyPickup();
    UPROPERTY(VisibleAnywhere) TObjectPtr<UStaticMeshComponent> Visual;
    UPROPERTY(VisibleAnywhere) TObjectPtr<UTextRenderComponent> Caption;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) EPickupKind Kind = EPickupKind::Energy;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) float RestoreAmount = 45.f;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) FName PersistentID;
    UFUNCTION(BlueprintCallable) void Configure(EPickupKind NewKind, FName NewID, float NewAmount = 45.f);
    virtual bool CanInteract_Implementation(APawn* Pawn) const override;
    virtual FText InteractionLabel_Implementation(APawn* Pawn) const override;
    virtual void Interact_Implementation(APawn* Pawn) override;
    UFUNCTION(BlueprintImplementableEvent) void OnCollected();
};
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/GreyGate.cpp

`````cpp
#include "Puzzles/GreyGate.h"
#include "Puzzles/PuzzleMechanism.h"
#include "Progression/GreyGameInstance.h"
#include "Components/StaticMeshComponent.h"
#include "Components/TextRenderComponent.h"
#include "UObject/ConstructorHelpers.h"
AGreyGate::AGreyGate()
{
    Barrier = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("GateBarrier"));
    SetRootComponent(Barrier);
    static ConstructorHelpers::FObjectFinder<UStaticMesh> Cube(TEXT("/Engine/BasicShapes/Cube.Cube"));
    if (Cube.Succeeded()) Barrier->SetStaticMesh(Cube.Object);
    Barrier->SetRelativeScale3D(FVector(0.75f,5.0f,3.1f));
    Barrier->SetCollisionProfileName(TEXT("BlockAll"));
    Caption = CreateDefaultSubobject<UTextRenderComponent>(TEXT("GateCaption"));
    Caption->SetupAttachment(Barrier);
    Caption->SetRelativeLocation(FVector(0,0,70));
    Caption->SetAbsolute(false,false,true);
    Caption->SetWorldSize(30.f);
    Caption->SetHorizontalAlignment(EHTA_Center);
    Caption->SetText(FText::FromString(TEXT("LOCKED")));
}
void AGreyGate::Configure(APuzzleMechanism* InSource, FName InShortcut)
{
    if (Source) Source->OnStateChanged.RemoveDynamic(this, &AGreyGate::SourceChanged);
    Source = InSource; ShortcutID = InShortcut;
    if (Source) Source->OnStateChanged.AddDynamic(this, &AGreyGate::SourceChanged);
    RefreshGate();
}
void AGreyGate::SourceChanged(APuzzleMechanism*, EPuzzleState, EPuzzleState) { RefreshGate(); }
void AGreyGate::RefreshGate()
{
    const UGreyGameInstance* GI = Cast<UGreyGameInstance>(GetGameInstance());
    const bool bShouldOpen = (Source && Source->IsTriggered()) || (GI && GI->HasShortcut(ShortcutID));
    if (bOpen || !bShouldOpen) return;
    bOpen = true;
    Barrier->SetCollisionEnabled(ECollisionEnabled::NoCollision);
    Barrier->SetVisibility(false, true);
    Caption->SetVisibility(false);
    OnGateOpened();
}
void AGreyGate::EndPlay(const EEndPlayReason::Type Reason)
{
    if (Source) Source->OnStateChanged.RemoveDynamic(this, &AGreyGate::SourceChanged);
    Super::EndPlay(Reason);
}
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/GreyGate.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Puzzles/PuzzleMechanism.h"
#include "GreyGate.generated.h"
class UStaticMeshComponent;
class UTextRenderComponent;
UCLASS(Blueprintable)
class THRESHOLDRUN_API AGreyGate : public AActor
{
    GENERATED_BODY()
public:
    AGreyGate();
    UPROPERTY(VisibleAnywhere) TObjectPtr<UStaticMeshComponent> Barrier;
    UPROPERTY(VisibleAnywhere) TObjectPtr<UTextRenderComponent> Caption;
    UPROPERTY(BlueprintReadOnly) bool bOpen = false;
    UFUNCTION(BlueprintCallable) void Configure(APuzzleMechanism* InSource, FName InShortcut);
    UFUNCTION(BlueprintCallable) void RefreshGate();
    UFUNCTION(BlueprintImplementableEvent) void OnGateOpened();
protected:
    virtual void EndPlay(const EEndPlayReason::Type Reason) override;
private:
    UPROPERTY() TObjectPtr<APuzzleMechanism> Source;
    UPROPERTY() FName ShortcutID;
    UFUNCTION() void SourceChanged(APuzzleMechanism* Mechanism, EPuzzleState Before, EPuzzleState After);
};
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/LogicRelay.cpp

`````cpp
#include "Puzzles/LogicRelay.h"
#include "Components/StaticMeshComponent.h"
ALogicRelay::ALogicRelay()
{
    Visual->SetRelativeScale3D(FVector(0.6f));
    SetCaptionText(TEXT("RELAY: SWITCH + PLATE"));
}
void ALogicRelay::Configure(const TArray<APuzzleMechanism*>& NewInputs, bool bAll)
{
    for (APuzzleMechanism* M : Inputs) if (IsValid(M)) M->OnStateChanged.RemoveDynamic(this, &ALogicRelay::InputChanged);
    Inputs.Empty(); bRequireAll = bAll;
    for (APuzzleMechanism* M : NewInputs)
        if (IsValid(M) && M != this && !Inputs.Contains(M))
        { Inputs.Add(M); M->OnStateChanged.AddDynamic(this, &ALogicRelay::InputChanged); }
    Reevaluate();
}
void ALogicRelay::InputChanged(APuzzleMechanism*, EPuzzleState, EPuzzleState) { Reevaluate(); }
void ALogicRelay::Reevaluate()
{
    if (IsTriggered() || Inputs.IsEmpty()) return;
    bool bSatisfied = bRequireAll;
    for (const APuzzleMechanism* M : Inputs)
    {
        const bool bReady = IsValid(M) && M->IsTriggered();
        if (bRequireAll) bSatisfied &= bReady;
        else bSatisfied |= bReady;
    }
    if (bSatisfied && TryActivate()) TryTrigger(); // latch permanently this run
}
void ALogicRelay::EndPlay(const EEndPlayReason::Type Reason)
{
    for (APuzzleMechanism* M : Inputs)
        if (IsValid(M)) M->OnStateChanged.RemoveDynamic(this, &ALogicRelay::InputChanged);
    Super::EndPlay(Reason);
}
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/LogicRelay.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "Puzzles/PuzzleMechanism.h"
#include "LogicRelay.generated.h"
UCLASS(Blueprintable)
class THRESHOLDRUN_API ALogicRelay : public APuzzleMechanism
{
    GENERATED_BODY()
public:
    ALogicRelay();
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Puzzle") bool bRequireAll = true;
    UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, Category="Puzzle") TArray<TObjectPtr<APuzzleMechanism>> Inputs;
    UFUNCTION(BlueprintCallable) void Configure(const TArray<APuzzleMechanism*>& NewInputs, bool bAll);
    UFUNCTION(BlueprintCallable) void Reevaluate();
protected:
    virtual void EndPlay(const EEndPlayReason::Type Reason) override;
private:
    UFUNCTION() void InputChanged(APuzzleMechanism* Source, EPuzzleState Before, EPuzzleState After);
};
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/PressurePlate.cpp

`````cpp
#include "Puzzles/PressurePlate.h"
#include "Components/StaticMeshComponent.h"
#include "Components/TextRenderComponent.h"
#include "Components/BoxComponent.h"
#include "GameFramework/Pawn.h"
#include "Components/PrimitiveComponent.h"
APressurePlate::APressurePlate()
{
    Visual->SetRelativeScale3D(FVector(1.7f,1.7f,0.15f));
    PressureArea = CreateDefaultSubobject<UBoxComponent>(TEXT("PressureArea"));
    PressureArea->SetupAttachment(Visual);
    PressureArea->SetRelativeLocation(FVector(0,0,80));
    PressureArea->SetBoxExtent(FVector(95,95,95));
    PressureArea->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
    PressureArea->SetCollisionResponseToAllChannels(ECR_Ignore);
    PressureArea->SetCollisionResponseToChannel(ECC_Pawn, ECR_Overlap);
    PressureArea->SetGenerateOverlapEvents(true);
    Caption->SetAbsolute(false,false,true);
    Caption->SetRelativeLocation(FVector(0,0,780));
    SetCaptionText(TEXT("PRESSURE PLATE"));
}
void APressurePlate::BeginPlay()
{
    Super::BeginPlay();
    PressureArea->OnComponentBeginOverlap.AddDynamic(this, &APressurePlate::OnEnter);
    PressureArea->OnComponentEndOverlap.AddDynamic(this, &APressurePlate::OnExit);
}
void APressurePlate::OnEnter(UPrimitiveComponent*, AActor* Actor, UPrimitiveComponent*, int32, bool, const FHitResult&)
{
    if (!Cast<APawn>(Actor)) return;
    const TWeakObjectPtr<AActor> Key(Actor);
    if (Pressers.Contains(Key)) return;
    Pressers.Add(Key);
    if (State == EPuzzleState::Ready) { TryActivate(); TryTrigger(); }
}
void APressurePlate::OnExit(UPrimitiveComponent*, AActor* Actor, UPrimitiveComponent*, int32)
{
    Pressers.Remove(TWeakObjectPtr<AActor>(Actor));
    if (Pressers.IsEmpty()) ResetMechanism();
}
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/PressurePlate.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "Puzzles/PuzzleMechanism.h"
#include "PressurePlate.generated.h"
class UBoxComponent;
UCLASS(Blueprintable)
class THRESHOLDRUN_API APressurePlate : public APuzzleMechanism
{
    GENERATED_BODY()
public:
    APressurePlate();
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly) TObjectPtr<UBoxComponent> PressureArea;
protected:
    virtual void BeginPlay() override;
private:
    UFUNCTION() void OnEnter(UPrimitiveComponent*, AActor*, UPrimitiveComponent*, int32, bool, const FHitResult&);
    UFUNCTION() void OnExit(UPrimitiveComponent*, AActor*, UPrimitiveComponent*, int32);
    TSet<TWeakObjectPtr<AActor>> Pressers;
};
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/PuzzleMechanism.cpp

`````cpp
#include "Puzzles/PuzzleMechanism.h"
#include "Components/StaticMeshComponent.h"
#include "Components/TextRenderComponent.h"
#include "UObject/ConstructorHelpers.h"
APuzzleMechanism::APuzzleMechanism()
{
    PrimaryActorTick.bCanEverTick = false;
    Visual = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Visual"));
    SetRootComponent(Visual);
    static ConstructorHelpers::FObjectFinder<UStaticMesh> Cube(TEXT("/Engine/BasicShapes/Cube.Cube"));
    if (Cube.Succeeded()) Visual->SetStaticMesh(Cube.Object);
    Visual->SetCollisionProfileName(TEXT("BlockAll"));
    Visual->SetRelativeScale3D(FVector(0.8f,0.8f,1.2f));
    Caption = CreateDefaultSubobject<UTextRenderComponent>(TEXT("Caption"));
    Caption->SetupAttachment(Visual);
    Caption->SetRelativeLocation(FVector(0.f,0.f,105.f));
    Caption->SetRelativeScale3D(FVector(1.25f));
    Caption->SetHorizontalAlignment(EHTA_Center);
    Caption->SetTextRenderColor(FColor::Cyan);
    Caption->SetWorldSize(22.f);
    Caption->SetText(FText::FromString(TEXT("Puzzle")));
}
void APuzzleMechanism::SetCaptionText(const FString& Text)
{ Caption->SetText(FText::FromString(Text)); }
void APuzzleMechanism::RefreshState()
{
    const EPuzzleState NewState = bTriggered ? EPuzzleState::Triggered : bLocked ? EPuzzleState::Locked
        : !bPowered ? EPuzzleState::Unpowered : bActivated ? EPuzzleState::Activated : EPuzzleState::Ready;
    if (NewState != State)
    {
        const EPuzzleState OldState = State;
        State = NewState;
        Caption->SetTextRenderColor(State == EPuzzleState::Triggered ? FColor::Green : FColor::Cyan);
        OnStateChanged.Broadcast(this, OldState, State);
    }
}
bool APuzzleMechanism::TryActivate()
{
    if (State != EPuzzleState::Ready) return false;
    bActivated = true; RefreshState(); return true;
}
bool APuzzleMechanism::TryTrigger()
{
    if (State != EPuzzleState::Activated) return false;
    bTriggered = true; RefreshState(); return true;
}
void APuzzleMechanism::ResetMechanism()
{ bTriggered = false; bActivated = false; RefreshState(); }
void APuzzleMechanism::SetPowered(bool Value)
{ bPowered = Value; if (!bPowered && !bTriggered) bActivated = false; RefreshState(); }
void APuzzleMechanism::SetLocked(bool Value)
{ bLocked = Value; if (bLocked && !bTriggered) bActivated = false; RefreshState(); }
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/PuzzleMechanism.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "PuzzleMechanism.generated.h"
class UStaticMeshComponent;
class UTextRenderComponent;
class APuzzleMechanism;
UENUM(BlueprintType)
enum class EPuzzleState : uint8 { Locked, Unpowered, Ready, Activated, Triggered };
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(FPuzzleStateChanged, APuzzleMechanism*, Mechanism, EPuzzleState, Before, EPuzzleState, After);
UCLASS(Blueprintable)
class THRESHOLDRUN_API APuzzleMechanism : public AActor
{
    GENERATED_BODY()
public:
    APuzzleMechanism();
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly) TObjectPtr<UStaticMeshComponent> Visual;
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly) TObjectPtr<UTextRenderComponent> Caption;
    UPROPERTY(BlueprintReadOnly, Category="Puzzle") EPuzzleState State = EPuzzleState::Ready;
    UPROPERTY(BlueprintAssignable, Category="Puzzle") FPuzzleStateChanged OnStateChanged;
    UFUNCTION(BlueprintCallable, Category="Puzzle") bool TryActivate();
    UFUNCTION(BlueprintCallable, Category="Puzzle") bool TryTrigger();
    UFUNCTION(BlueprintCallable, Category="Puzzle") void ResetMechanism();
    UFUNCTION(BlueprintCallable, Category="Puzzle") void SetPowered(bool bValue);
    UFUNCTION(BlueprintCallable, Category="Puzzle") void SetLocked(bool bValue);
    UFUNCTION(BlueprintCallable) void SetCaptionText(const FString& Text);
    UFUNCTION(BlueprintPure) bool IsTriggered() const { return State == EPuzzleState::Triggered; }
protected:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Puzzle") bool bLocked = false;
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Puzzle") bool bPowered = true;
    bool bActivated = false;
    bool bTriggered = false;
    void RefreshState();
};
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/SwitchMechanism.cpp

`````cpp
#include "Puzzles/SwitchMechanism.h"
#include "Survival/SurvivalVitals.h"
ASwitchMechanism::ASwitchMechanism() { SetCaptionText(TEXT("SWITCH [E]")); }
bool ASwitchMechanism::CanInteract_Implementation(APawn* Pawn) const
{
    const USurvivalVitals* V = Pawn ? Pawn->FindComponentByClass<USurvivalVitals>() : nullptr;
    return State == EPuzzleState::Ready && V && !V->IsDead() && V->Energy >= EnergyCost;
}
FText ASwitchMechanism::InteractionLabel_Implementation(APawn*) const
{
    if (IsTriggered()) return FText::FromString(TEXT("Switch activated"));
    return FText::FromString(FString::Printf(TEXT("Press E: activate switch (%.0f energy)"), EnergyCost));
}
void ASwitchMechanism::Interact_Implementation(APawn* Pawn)
{
    if (!CanInteract_Implementation(Pawn)) return;
    USurvivalVitals* V = Pawn->FindComponentByClass<USurvivalVitals>();
    if (V && V->SpendEnergy(EnergyCost) && TryActivate() && TryTrigger()) OnSwitchUsed();
}
`````

## File: ThresholdRun/Source/ThresholdRun/Puzzles/SwitchMechanism.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "Puzzles/PuzzleMechanism.h"
#include "Interaction/PrototypeInteractable.h"
#include "SwitchMechanism.generated.h"
UCLASS(Blueprintable)
class THRESHOLDRUN_API ASwitchMechanism : public APuzzleMechanism, public IPrototypeInteractable
{
    GENERATED_BODY()
public:
    ASwitchMechanism();
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Puzzle") float EnergyCost = 12.f;
    virtual bool CanInteract_Implementation(APawn* Pawn) const override;
    virtual FText InteractionLabel_Implementation(APawn* Pawn) const override;
    virtual void Interact_Implementation(APawn* Pawn) override;
    UFUNCTION(BlueprintImplementableEvent, Category="Puzzle") void OnSwitchUsed();
};
`````

## File: ThresholdRun/Source/ThresholdRun/Survival/ExposureZone.cpp

`````cpp
#include "Survival/ExposureZone.h"
#include "Survival/SurvivalVitals.h"
#include "Components/BoxComponent.h"
AExposureZone::AExposureZone()
{
    Trigger = CreateDefaultSubobject<UBoxComponent>(TEXT("HazardTrigger"));
    SetRootComponent(Trigger);
    Trigger->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
    Trigger->SetCollisionResponseToAllChannels(ECR_Ignore);
    Trigger->SetCollisionResponseToChannel(ECC_Pawn, ECR_Overlap);
    Trigger->SetGenerateOverlapEvents(true);
}
void AExposureZone::BeginPlay()
{
    Super::BeginPlay(); Trigger->SetBoxExtent(BoxExtent);
    Trigger->OnComponentBeginOverlap.AddDynamic(this, &AExposureZone::Enter);
    Trigger->OnComponentEndOverlap.AddDynamic(this, &AExposureZone::Exit);
}
void AExposureZone::Enter(UPrimitiveComponent*, AActor* Actor, UPrimitiveComponent*, int32, bool, const FHitResult&)
{
    if (USurvivalVitals* V = Actor ? Actor->FindComponentByClass<USurvivalVitals>() : nullptr)
    { if (!Occupants.Contains(V)) { Occupants.Add(V); V->AddExposure(OxygenDrainPerSecond, WarmthDrainPerSecond); } }
}
void AExposureZone::Exit(UPrimitiveComponent*, AActor* Actor, UPrimitiveComponent*, int32)
{
    if (USurvivalVitals* V = Actor ? Actor->FindComponentByClass<USurvivalVitals>() : nullptr)
    { if (Occupants.Remove(V) > 0) V->RemoveExposure(OxygenDrainPerSecond, WarmthDrainPerSecond); }
}
void AExposureZone::EndPlay(const EEndPlayReason::Type Reason)
{
    for (USurvivalVitals* V : Occupants)
        if (IsValid(V)) V->RemoveExposure(OxygenDrainPerSecond, WarmthDrainPerSecond);
    Occupants.Empty(); Super::EndPlay(Reason);
}
`````

## File: ThresholdRun/Source/ThresholdRun/Survival/ExposureZone.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "ExposureZone.generated.h"
class UBoxComponent;
class USurvivalVitals;
UCLASS(Blueprintable)
class THRESHOLDRUN_API AExposureZone : public AActor
{
    GENERATED_BODY()
public:
    AExposureZone();
    UPROPERTY(VisibleAnywhere) TObjectPtr<UBoxComponent> Trigger;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) float OxygenDrainPerSecond = 0.f;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) float WarmthDrainPerSecond = 11.f;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) FVector BoxExtent = FVector(420.f, 450.f, 150.f);
    virtual void BeginPlay() override;
private:
    UFUNCTION() void Enter(UPrimitiveComponent*, AActor*, UPrimitiveComponent*, int32, bool, const FHitResult&);
    UFUNCTION() void Exit(UPrimitiveComponent*, AActor*, UPrimitiveComponent*, int32);
    UPROPERTY() TArray<TObjectPtr<USurvivalVitals>> Occupants;
    virtual void EndPlay(const EEndPlayReason::Type Reason) override;
};
`````

## File: ThresholdRun/Source/ThresholdRun/Survival/SurvivalVitals.cpp

`````cpp
#include "Survival/SurvivalVitals.h"
USurvivalVitals::USurvivalVitals() { PrimaryComponentTick.bCanEverTick = true; }
void USurvivalVitals::TickComponent(float Delta, ELevelTick TickType, FActorComponentTickFunction* Tick)
{
    Super::TickComponent(Delta, TickType, Tick);
    if (bDead) return;
    const bool bActuallySprint = bSprinting && Energy > 0.f;
    const float EnergyDelta = bActuallySprint ? -(BaseEnergyDrainPerSecond + SprintEnergyDrainPerSecond) * Delta
                                             : (EnergyRecoveryPerSecond - BaseEnergyDrainPerSecond) * Delta;
    Energy = FMath::Clamp(Energy + EnergyDelta, 0.f, MaxValue);
    Oxygen = FMath::Clamp(Oxygen - OxygenExposure * Delta, 0.f, MaxValue);
    Warmth = FMath::Clamp(Warmth - WarmthExposure * Delta, 0.f, MaxValue);
    CheckDeath();
}
bool USurvivalVitals::SpendEnergy(float Amount)
{
    if (Amount < 0.f || bDead || Energy < Amount) return false;
    Energy -= Amount; CheckDeath(); return true;
}
void USurvivalVitals::Restore(float E, float O, float W)
{
    if (bDead) return;
    Energy = FMath::Clamp(Energy + E, 0.f, MaxValue);
    Oxygen = FMath::Clamp(Oxygen + O, 0.f, MaxValue);
    Warmth = FMath::Clamp(Warmth + W, 0.f, MaxValue);
}
void USurvivalVitals::AddExposure(float O, float W)
{
    OxygenExposure += FMath::Max(O, 0.f);
    WarmthExposure += FMath::Max(W, 0.f);
}
void USurvivalVitals::RemoveExposure(float O, float W)
{
    OxygenExposure = FMath::Max(0.f, OxygenExposure - FMath::Max(O, 0.f));
    WarmthExposure = FMath::Max(0.f, WarmthExposure - FMath::Max(W, 0.f));
}
void USurvivalVitals::CheckDeath()
{
    if (!bDead && (Energy <= 0.f || Oxygen <= 0.f || Warmth <= 0.f))
    { bDead = true; OnDied.Broadcast(); }
}
`````

## File: ThresholdRun/Source/ThresholdRun/Survival/SurvivalVitals.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "SurvivalVitals.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE(FVitalsDeath);
UCLASS(ClassGroup=(Survival), meta=(BlueprintSpawnableComponent))
class THRESHOLDRUN_API USurvivalVitals : public UActorComponent
{
    GENERATED_BODY()
public:
    USurvivalVitals();
    virtual void TickComponent(float Delta, ELevelTick TickType, FActorComponentTickFunction* Tick) override;
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Vitals") float MaxValue = 100.f;
    UPROPERTY(BlueprintReadOnly, Category="Vitals") float Energy = 100.f;
    UPROPERTY(BlueprintReadOnly, Category="Vitals") float Oxygen = 100.f;
    UPROPERTY(BlueprintReadOnly, Category="Vitals") float Warmth = 100.f;
    UPROPERTY(EditAnywhere, Category="Vitals") float BaseEnergyDrainPerSecond = 0.35f;
    UPROPERTY(EditAnywhere, Category="Vitals") float SprintEnergyDrainPerSecond = 4.0f;
    UPROPERTY(EditAnywhere, Category="Vitals") float EnergyRecoveryPerSecond = 1.75f;
    UPROPERTY(BlueprintAssignable) FVitalsDeath OnDied;
    UFUNCTION(BlueprintCallable) bool SpendEnergy(float Amount);
    UFUNCTION(BlueprintCallable) void Restore(float EnergyDelta, float OxygenDelta, float WarmthDelta);
    UFUNCTION(BlueprintCallable) void AddExposure(float OxygenDrainPerSecond, float WarmthDrainPerSecond);
    UFUNCTION(BlueprintCallable) void RemoveExposure(float OxygenDrainPerSecond, float WarmthDrainPerSecond);
    UFUNCTION(BlueprintPure) bool CanSprint() const { return Energy > 5.f && !bDead; }
    UFUNCTION(BlueprintCallable) void SetSprinting(bool bValue) { bSprinting = bValue; }
    UFUNCTION(BlueprintPure) bool IsDead() const { return bDead; }
private:
    bool bSprinting = false;
    bool bDead = false;
    float OxygenExposure = 0.f;
    float WarmthExposure = 0.f;
    void CheckDeath();
};
`````

## File: ThresholdRun/Source/ThresholdRun/Tests/ThresholdSmokeTests.cpp

`````cpp
// Runtime module: available to Development games, including packaged Win64.
// No test actor is placed in the map; this latent command survives OpenLevel.
#include "CoreMinimal.h"

#if WITH_DEV_AUTOMATION_TESTS
#include "Misc/AutomationTest.h"
#include "Misc/CommandLine.h"
#include "Misc/Parse.h"
#include "Misc/Paths.h"
#include "Misc/FileHelper.h"
#include "HAL/PlatformTime.h"
#include "Engine/Engine.h"
#include "Engine/World.h"
#include "EngineUtils.h"
#include "Kismet/GameplayStatics.h"
#include "Camera/CameraComponent.h"
#include "Components/StaticMeshComponent.h"
#include "GameFramework/PlayerController.h"
#include "Player/GreyPlayer.h"
#include "Survival/SurvivalVitals.h"
#include "Puzzles/SwitchMechanism.h"
#include "Puzzles/GreyGate.h"
#include "Progression/GreyGameMode.h"
#include "Progression/GreyGameInstance.h"
#include "Progression/GreyMetaSave.h"

namespace ThresholdSmoke
{
const FString Slot = TEXT("ThresholdRunMeta_v1");
const FName Shortcut(TEXT("shortcut_gate_1"));
const FName Blueprint(TEXT("blueprint_thermal_lens"));

UWorld* FindWorld()
{
    if (GEngine)
        for (const FWorldContext& Context : GEngine->GetWorldContexts())
            if (Context.WorldType == EWorldType::Game && Context.World()) return Context.World();
    return nullptr;
}

template<class T> T* FindAtX(UWorld* World, float X)
{
    for (TActorIterator<T> It(World); It; ++It)
        if (FMath::Abs(It->GetActorLocation().X - X) < 1.f) return *It;
    return nullptr;
}

bool IsOpen(const AGreyGate* Gate)
{
    return Gate && Gate->bOpen && Gate->Barrier
        && Gate->Barrier->GetCollisionEnabled() == ECollisionEnabled::NoCollision
        && !Gate->Barrier->IsVisible();
}

class FGameLoop final : public IAutomationLatentCommand
{
    enum class EStep { Startup, Gate, Death, Reload };
    FAutomationTestBase* Test;
    EStep Step = EStep::Startup;
    double Deadline = FPlatformTime::Seconds() + 30.0;
    double SwitchStarted = 0;
    bool bRestart = false;
    FString Root;
    TWeakObjectPtr<UWorld> OriginalWorld;
    TWeakObjectPtr<AGreyPlayer> OriginalPlayer;
    TWeakObjectPtr<AGreyGate> Gate;

    bool Fail(const FString& Why) { Test->AddError(Why); return true; }
    bool Check(bool Value, const TCHAR* Why)
    {
        if (!Value) Test->AddError(Why);
        return Value;
    }
    bool VitalsOK(AGreyPlayer* Player)
    {
        const USurvivalVitals* V = Player ? Player->Vitals.Get() : nullptr;
        return V && V->IsRegistered() && !V->IsDead()
            && FMath::IsNearlyEqual(V->MaxValue, 100.f)
            && FMath::IsNearlyEqual(V->Energy, 100.f, 0.01f)
            && FMath::IsNearlyEqual(V->Oxygen, 100.f, 0.01f)
            && FMath::IsNearlyEqual(V->Warmth, 100.f, 0.01f);
    }
    bool VerifyMeta(const UGreyMetaSave* Meta, int32 Runs, int32 Seed)
    {
        return Check(Meta && Meta->RunsCompleted == Runs && Meta->Victories == 0
            && Meta->NextSeed == Seed && Meta->Shortcuts.Num() == 1
            && Meta->Shortcuts.Contains(Shortcut) && Meta->Blueprints.Num() == 1
            && Meta->Blueprints.Contains(Blueprint) && Meta->Knowledge.IsEmpty(),
            TEXT("SaveGame shortcut/blueprint, run count, victory count or seed mismatch"));
    }
    bool VerifyDisk(int32 Runs, int32 Seed)
    {
        // Load a new object from disk, independent of the GameInstance cache.
        return VerifyMeta(Cast<UGreyMetaSave>(
            UGameplayStatics::LoadGameFromSlot(Slot, 0)), Runs, Seed);
    }
    bool VerifyTransientReset(UWorld* World)
    {
        const ASwitchMechanism* Second = FindAtX<ASwitchMechanism>(World, 2520.f);
        const AGreyGate* SecondGate = FindAtX<AGreyGate>(World, 3300.f);
        return Check(Second && Second->State == EPuzzleState::Ready
            && SecondGate && !SecondGate->bOpen,
            TEXT("Second puzzle must start ready with gate 2 closed"));
    }
    bool Receipt(const TCHAR* Name)
    {
        return Check(FFileHelper::SaveStringToFile(TEXT("PASS\n"),
            *FPaths::Combine(Root, Name)), TEXT("Could not write smoke receipt"));
    }
public:
    explicit FGameLoop(FAutomationTestBase* InTest) : Test(InTest) {}
    bool Update() override
    {
        UWorld* W = FindWorld();
        const double Now = FPlatformTime::Seconds();
        if (Now > Deadline) return Fail(FString::Printf(TEXT("Smoke timeout in phase %d"), int32(Step)));
        if (!W || !W->HasBegunPlay()) return false;
        AGreyPlayer* Player = Cast<AGreyPlayer>(UGameplayStatics::GetPlayerPawn(W, 0));
        UGreyGameInstance* GI = Cast<UGreyGameInstance>(W->GetGameInstance());

        if (Step == EStep::Startup)
        {
            // Refuse writes unless the runner has routed the actual Saved directory
            // into a dedicated sandbox. Production slot names remain unchanged.
            FString UserDir;
            if (!FParse::Value(FCommandLine::Get(), TEXT("ThresholdSmokeRoot="), Root)
                || !FParse::Value(FCommandLine::Get(), TEXT("UserDir="), UserDir)
                || !FParse::Param(FCommandLine::Get(), TEXT("SaveToUserDir")))
                return Fail(TEXT("Use Tools/Run-SmokeTests.ps1: isolated UserDir is required"));
            Root = FPaths::ConvertRelativePathToFull(Root);
            UserDir = FPaths::ConvertRelativePathToFull(UserDir);
            FPaths::NormalizeDirectoryName(Root);
            FPaths::NormalizeDirectoryName(UserDir);
            if (!FPaths::IsSamePath(Root, UserDir)
                || !FPaths::IsUnderDirectory(FPaths::ConvertRelativePathToFull(FPaths::ProjectSavedDir()), Root))
                return Fail(TEXT("Actual Saved directory is outside the requested smoke sandbox"));
            if (UGameplayStatics::GetCurrentLevelName(W, true) != TEXT("L_Greybox")
                || !Cast<AGreyGameMode>(W->GetAuthGameMode()) || !GI || !GI->Meta)
                return Fail(TEXT("Expected L_Greybox with GreyGameMode and GreyGameInstance"));
            if (!Player || !Player->HasActorBegunPlay() || !Player->GetController()) return false;
            if (!Check(VitalsOK(Player), TEXT("TC1: player/vitals did not initialize to 100,100,100"))) return true;
            Test->AddInfo(TEXT("TC1 PASS: possessed AGreyPlayer and initialized SurvivalVitals"));
            AGreyGate* FirstGate = FindAtX<AGreyGate>(W, 1100.f);
            ASwitchMechanism* FirstSwitch = FindAtX<ASwitchMechanism>(W, 430.f);
            if (!FirstGate || !FirstSwitch) return Fail(TEXT("Generated gate 1 / switch 1 missing"));
            bRestart = FParse::Param(FCommandLine::Get(), TEXT("ThresholdSmokeRestart"));
            if (bRestart)
            {
                if (!Check(FPaths::FileExists(FPaths::Combine(Root, TEXT("Fresh.ok"))),
                        TEXT("Restart check requires a successful fresh-process run"))
                    || !VerifyMeta(GI->Meta.Get(), 1, 20264) || !VerifyDisk(1, 20264)
                    || !Check(GI->GetRunSeed() == 20264 && GI->GetRunNumber() == 2,
                        TEXT("Restart did not initialize saved seed/run number"))
                    || !Check(IsOpen(FirstGate) && FirstSwitch->State == EPuzzleState::Ready,
                        TEXT("Restart must apply saved shortcut without retaining switch state"))) return true;
                Test->AddInfo(TEXT("RESTART PASS: new GameInstance loaded shortcuts/blueprints from disk"));
                if (VerifyTransientReset(W)) Receipt(TEXT("Restart.ok"));
                return true;
            }
            if (!Check(!UGameplayStatics::DoesSaveGameExist(Slot, 0)
                    && GI->Meta->Shortcuts.IsEmpty() && GI->Meta->Blueprints.IsEmpty()
                    && GI->Meta->Knowledge.IsEmpty() && GI->Meta->RunsCompleted == 0
                    && GI->Meta->Victories == 0 && GI->GetRunSeed() == 12345,
                    TEXT("Fresh smoke test requires an empty sandbox"))
                || !Check(!FirstGate->bOpen && FirstGate->Barrier
                    && FirstGate->Barrier->GetCollisionEnabled() != ECollisionEnabled::NoCollision
                    && FirstSwitch->State == EPuzzleState::Ready
                    && FMath::IsNearlyEqual(FirstSwitch->EnergyCost, 12.f),
                    TEXT("TC2: first switch/gate must start ready/closed with 12 energy cost"))) return true;

            // Position and aim the player, then exercise the production camera trace
            // and TryInteract entry point. This does not synthesize keyboard input.
            Player->SetActorLocation(FirstSwitch->GetActorLocation() + FVector(-180, 0, 23),
                false, nullptr, ETeleportType::TeleportPhysics);
            APlayerController* PC = Cast<APlayerController>(Player->GetController());
            if (!PC || !Player->Camera) return Fail(TEXT("Player camera/controller missing"));
            const FRotator Aim = (FirstSwitch->GetActorLocation() - Player->Camera->GetComponentLocation()).Rotation();
            PC->SetControlRotation(Aim);
            Player->Camera->SetWorldRotation(Aim);
            if (!Check(Player->FindInteractable() == FirstSwitch,
                    TEXT("TC2: camera raycast did not hit first switch"))) return true;
            const float EnergyBefore = Player->Vitals->Energy;
            SwitchStarted = FPlatformTime::Seconds();
            Player->TryInteract();
            if (!Check(FirstSwitch->IsTriggered()
                    && FMath::IsNearlyEqual(Player->Vitals->Energy, EnergyBefore - 12.f, 0.01f),
                    TEXT("TC2: switch interaction did not trigger/spend 12 energy"))) return true;
            OriginalWorld = W;
            OriginalPlayer = Player;
            Gate = FirstGate;
            Step = EStep::Gate;
            Deadline = SwitchStarted + 2.0;
            // Observe synchronous opening now, rather than penalizing a later slow tick.
            if (!IsOpen(FirstGate)) return false;
        }
        if (Step == EStep::Gate)
        {
            if (FPlatformTime::Seconds() > Deadline) return Fail(TEXT("TC2: gate missed 2-second deadline"));
            if (!IsOpen(Gate.Get())) return false;
            Test->AddInfo(TEXT("TC2 PASS: switch opened gate with visibility/collision disabled within 2s"));
            // Exercise real production persistence, not a test-owned SaveGame object.
            if (!Player || !GI) return Fail(TEXT("TC3: player or GameInstance disappeared"));
            if (!Check(GI->UnlockShortcut(Shortcut) && GI->UnlockBlueprint(Blueprint),
                    TEXT("TC3: could not create permanent unlock fixture")) || !VerifyDisk(0, 12345)) return true;
            // Restore only spent energy. Oxygen death must come from the normal tick
            // -> OnDied -> AGreyGameMode::EndRun(false) -> delayed OpenLevel chain.
            Player->Vitals->Restore(100.f, 0.f, 0.f);
            Player->Vitals->AddExposure(10000.f, 0.f);
            Step = EStep::Death;
            Deadline = FPlatformTime::Seconds() + 5.0;
            return false;
        }
        if (Step == EStep::Death)
        {
            AGreyPlayer* Old = OriginalPlayer.Get();
            if (!Old) return Fail(TEXT("TC3: player disappeared before oxygen death was observed"));
            if (!Old->Vitals->IsDead()) return false;
            if (!Check(Old->Vitals->Oxygen <= 0.f && Old->Vitals->Energy > 0.f && Old->Vitals->Warmth > 0.f,
                    TEXT("TC3: death was not caused by oxygen reaching zero"))) return true;
            Step = EStep::Reload;
            Deadline = FPlatformTime::Seconds() + 30.0;
            return false;
        }
        if (W == OriginalWorld.Get()) return false;
        if (!Player || !Player->HasActorBegunPlay() || !Player->GetController()) return false;
        if (!Check(UGameplayStatics::GetCurrentLevelName(W, true) == TEXT("L_Greybox")
                && Cast<AGreyGameMode>(W->GetAuthGameMode())
                && Player != OriginalPlayer.Get() && VitalsOK(Player),
                TEXT("TC3: new map/player/vitals initialization incorrect"))) return true;
        if (!GI) return Fail(TEXT("TC3: reloaded GameInstance missing"));
        if (!VerifyMeta(GI->Meta.Get(), 1, 20264) || !VerifyDisk(1, 20264)
            || !Check(GI->GetRunSeed() == 20264 && GI->GetRunNumber() == 2,
                TEXT("TC3: death must advance seed and run number"))) return true;
        const AGreyGate* ReloadedGate = FindAtX<AGreyGate>(W, 1100.f);
        const ASwitchMechanism* ReloadedSwitch = FindAtX<ASwitchMechanism>(W, 430.f);
        if (!Check(IsOpen(ReloadedGate) && ReloadedSwitch && ReloadedSwitch->State == EPuzzleState::Ready,
                TEXT("TC3: permanent shortcut not applied or transient switch progress retained"))) return true;
        Test->AddInfo(TEXT("TC3 PASS: oxygen death reloaded map; disk and applied shortcuts/blueprints retained"));
        if (VerifyTransientReset(W)) Receipt(TEXT("Fresh.ok"));
        return true;
    }
};
}

IMPLEMENT_SIMPLE_AUTOMATION_TEST(FThresholdGameLoop, "ThresholdRun.Smoke.GameLoop",
    EAutomationTestFlags::ClientContext | EAutomationTestFlags::EngineFilter)
bool FThresholdGameLoop::RunTest(const FString& Parameters)
{
    ADD_LATENT_AUTOMATION_COMMAND(ThresholdSmoke::FGameLoop(this));
    return true;
}
#endif
`````

## File: ThresholdRun/Source/ThresholdRun/ThresholdRun.Build.cs

`````csharp
using UnrealBuildTool;
public class ThresholdRun : ModuleRules
{
    public ThresholdRun(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
        PublicDependencyModuleNames.AddRange(new string[] {
            "Core", "CoreUObject", "Engine", "InputCore", "EnhancedInput"
        });
    }
}
`````

## File: ThresholdRun/Source/ThresholdRun/ThresholdRun.cpp

`````cpp
#include "Modules/ModuleManager.h"
IMPLEMENT_PRIMARY_GAME_MODULE(FDefaultGameModuleImpl, ThresholdRun, "ThresholdRun");
`````

## File: ThresholdRun/Source/ThresholdRun/UI/GreyHUD.cpp

`````cpp
#include "UI/GreyHUD.h"
#include "Player/GreyPlayer.h"
#include "Progression/GreyGameInstance.h"
#include "Survival/SurvivalVitals.h"
#include "Engine/Canvas.h"
#include "Engine/Engine.h"
#include "GameFramework/PlayerController.h"
void AGreyHUD::DrawHUD()
{
    Super::DrawHUD();
    if (!Canvas || !GEngine || !PlayerOwner) return;
    const float X=Canvas->SizeX*0.5f, Y=Canvas->SizeY*0.5f;
    DrawLine(X-8,Y,X+8,Y,FLinearColor::White,1.5f);
    DrawLine(X,Y-8,X,Y+8,FLinearColor::White,1.5f);
    AGreyPlayer* P=Cast<AGreyPlayer>(PlayerOwner->GetPawn());
    UGreyGameInstance* GI=Cast<UGreyGameInstance>(GetGameInstance());
    if (GI)
    {
        DrawText(FString::Printf(TEXT("THRESHOLD RUN  |  Run %d  |  Seed %d"),GI->GetRunNumber(),GI->GetRunSeed()), FLinearColor::White,20,20,GEngine->GetSmallFont(),1.35f);
        DrawText(GI->GetRunMessage(),FLinearColor::Yellow,20,48,GEngine->GetSmallFont(),1.05f);
    }
    if (!P || !P->Vitals) return;
    const USurvivalVitals* V=P->Vitals;
    DrawText(FString::Printf(TEXT("ENERGY %.0f    OXYGEN %.0f    WARMTH %.0f"),V->Energy,V->Oxygen,V->Warmth),FLinearColor::Green,20,85,GEngine->GetSmallFont(),1.4f);
    DrawText(TEXT("WASD move  |  Mouse look  |  Shift sprint  |  Ctrl crouch  |  Space jump  |  E interact"),FLinearColor::White,20,118,GEngine->GetSmallFont(),1.05f);
    const float PX=P->GetActorLocation().X;
    const TCHAR* Objective=PX < 1250 ? TEXT("OBJECTIVE: Find gate 1 switch, then go east.")
                              : PX < 3450 ? TEXT("OBJECTIVE: Activate switch 2 AND stand on its pressure plate.")
                                          : TEXT("OBJECTIVE: Survive the cold zone and use the EXIT.");
    DrawText(Objective,FLinearColor::Yellow,20,150,GEngine->GetSmallFont(),1.15f);
    const FString Prompt=P->GetInteractionPrompt().ToString();
    if (!Prompt.IsEmpty())
        DrawText(Prompt,FLinearColor::Yellow,X-230,Y+30,GEngine->GetSmallFont(),1.45f);
}
`````

## File: ThresholdRun/Source/ThresholdRun/UI/GreyHUD.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/HUD.h"
#include "GreyHUD.generated.h"
UCLASS(Blueprintable)
class THRESHOLDRUN_API AGreyHUD : public AHUD
{
    GENERATED_BODY()
public:
    virtual void DrawHUD() override;
};
`````

## File: ThresholdRun/Source/ThresholdRun/World/GreyWorldBuilder.cpp

`````cpp
#include "World/GreyWorldBuilder.h"
#include "Puzzles/SwitchMechanism.h"
#include "Puzzles/PressurePlate.h"
#include "Puzzles/LogicRelay.h"
#include "Puzzles/GreyGate.h"
#include "Progression/GreyPickup.h"
#include "Progression/GreyExit.h"
#include "Progression/GreyGameInstance.h"
#include "Survival/ExposureZone.h"
#include "Engine/StaticMeshActor.h"
#include "Engine/TextRenderActor.h"
#include "Components/StaticMeshComponent.h"
#include "Components/TextRenderComponent.h"
#include "Engine/World.h"
#include "Engine/PointLight.h"
#include "Components/PointLightComponent.h"

AGreyWorldBuilder::AGreyWorldBuilder()
{
    PrimaryActorTick.bCanEverTick=false;
    // Visual Blueprints are optional. The native classes form a functioning fallback.
}
AActor* AGreyWorldBuilder::Block(const FVector& Center,const FVector& Dimensions)
{
    FActorSpawnParameters P;
    P.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;
    AStaticMeshActor* A=GetWorld()->SpawnActor<AStaticMeshActor>(Center,FRotator::ZeroRotator,P);
    if (!A) return nullptr;
    UStaticMeshComponent* M=A->GetStaticMeshComponent();
    M->SetMobility(EComponentMobility::Movable);
    static UStaticMesh* Cube=LoadObject<UStaticMesh>(nullptr,TEXT("/Engine/BasicShapes/Cube.Cube"));
    if (Cube) M->SetStaticMesh(Cube);
    M->SetWorldScale3D(Dimensions/100.f);
    M->SetCollisionProfileName(TEXT("BlockAll"));
    return A;
}
void AGreyWorldBuilder::Sign(const FString& Text,const FVector& Location,const FColor& Color)
{
    ATextRenderActor* T=GetWorld()->SpawnActor<ATextRenderActor>(Location,FRotator(0,180,0));
    if (!T) return;
    UTextRenderComponent* C=T->GetTextRender();
    C->SetText(FText::FromString(Text)); C->SetTextRenderColor(Color);
    C->SetWorldSize(42.f); C->SetHorizontalAlignment(EHTA_Center);
}
void AGreyWorldBuilder::MakeRoom(int32 Slot,bool bBranch)
{
    const float X=Slot*2200.f;
    Block(FVector(X,0,-50),FVector(1900,1400,100));
    // Long walls. Slot 1 makes a north-wall portal only if its optional branch is present.
    Block(FVector(X,-700,165),FVector(1900,70,330));
    if (Slot==1 && bBranch)
    {
        Block(FVector(X-600,700,165),FVector(700,70,330));
        Block(FVector(X+600,700,165),FVector(700,70,330));
    }
    else Block(FVector(X,700,165),FVector(1900,70,330));
    for (int32 Side=-1;Side<=1;Side+=2)
    {
        const bool bOpening=(Side>0 && Slot<2)||(Side<0 && Slot>0);
        const float SideX=X+Side*950.f;
        if (!bOpening) Block(FVector(SideX,0,165),FVector(70,1400,330));
        else
        {
            Block(FVector(SideX,-475,165),FVector(70,450,330));
            Block(FVector(SideX,475,165),FVector(70,450,330));
        }
    }
}
void AGreyWorldBuilder::MakeOptionalBranch()
{
    const float X=2200.f;
    Block(FVector(X,905,-50),FVector(500,410,100)); // connecting passage
    Block(FVector(X-250,905,165),FVector(70,410,330));
    Block(FVector(X+250,905,165),FVector(70,410,330));
    Block(FVector(X,1500,-50),FVector(850,800,100));
    Block(FVector(X-425,1500,165),FVector(70,800,330));
    Block(FVector(X+425,1500,165),FVector(70,800,330));
    Block(FVector(X,1900,165),FVector(850,70,330));
    Block(FVector(X-337.5f,1100,165),FVector(175,70,330));
    Block(FVector(X+337.5f,1100,165),FVector(175,70,330));
    Sign(TEXT("OPTIONAL CACHE"),FVector(X,1700,220),FColor::Yellow);
    if (AGreyPickup* P=SpawnPiece<AGreyPickup>(PickupClass,FVector(X,1550,55)))
        P->Configure(EPickupKind::Energy,NAME_None,45.f);
}
void AGreyWorldBuilder::BeginPlay()
{
    Super::BeginPlay();
    const UGreyGameInstance* GI=Cast<UGreyGameInstance>(GetGameInstance());
    FRandomStream Random(GI ? GI->GetRunSeed() : 12345);
    const bool bBranch=bAllowOptionalBranch && Random.FRand() > 0.36f;
    for(int32 I=0; I<3; ++I)
    {
        MakeRoom(I,bBranch);
        // An Empty Level has no light actors. Spawn lights to keep the prototype visible.
        if (APointLight* Light=GetWorld()->SpawnActor<APointLight>(FVector(I*2200.f,0,420),FRotator::ZeroRotator))
        {
            Light->GetPointLightComponent()->SetIntensity(45000.f);
            Light->GetPointLightComponent()->SetAttenuationRadius(2400.f);
        }
    }
    if (bBranch) MakeOptionalBranch();
    for (int32 I=0;I<2;++I)
    {
        const float CX=1100.f+I*2200.f;
        Block(FVector(CX,0,-50),FVector(300,500,100));
        Block(FVector(CX,-250,165),FVector(300,70,330));
        Block(FVector(CX,250,165),FVector(300,70,330));
    }
    Sign(TEXT("HUB 1: FURNACE / POWER SWITCH"),FVector(100,0,290));
    Sign(TEXT("HUB 2: PLATE + SWITCH = POWER"),FVector(1850,0,290));
    Sign(TEXT("HUB 3: COLD HAZARD / EXIT"),FVector(4020,0,290));
    const float Offset=Random.RandRange(-100,100);
    ASwitchMechanism* First=SpawnPiece<ASwitchMechanism>(SwitchClass,FVector(430,Offset-240,65));
    if (First) { First->SetCaptionText(TEXT("GATE 1 POWER SWITCH")); First->EnergyCost=12.f; }
    AGreyGate* Gate1=SpawnPiece<AGreyGate>(GateClass,FVector(1100,0,155));
    if (Gate1) Gate1->Configure(First,TEXT("shortcut_gate_1"));
    if (AGreyPickup* Shortcut=SpawnPiece<AGreyPickup>(PickupClass,FVector(1575,-360,55)))
        Shortcut->Configure(EPickupKind::Shortcut,TEXT("shortcut_gate_1"));
    if (AGreyPickup* Blueprint=SpawnPiece<AGreyPickup>(PickupClass,FVector(2140,-420,55)))
        Blueprint->Configure(EPickupKind::Blueprint,TEXT("blueprint_thermal_lens"));
    ASwitchMechanism* Second=SpawnPiece<ASwitchMechanism>(SwitchClass,FVector(2520,-265,65));
    if (Second) { Second->EnergyCost=15.f; Second->SetCaptionText(TEXT("GATE 2 POWER SWITCH")); }
    APressurePlate* Plate=SpawnPiece<APressurePlate>(PlateClass,FVector(2580,240,8));
    ALogicRelay* Relay=GetWorld()->SpawnActor<ALogicRelay>(FVector(2850,0,80),FRotator::ZeroRotator);
    if (Relay && Second && Plate)
    {
        TArray<APuzzleMechanism*> Inputs; Inputs.Add(Second); Inputs.Add(Plate);
        Relay->Configure(Inputs,true);
    }
    AGreyGate* Gate2=SpawnPiece<AGreyGate>(GateClass,FVector(3300,0,155));
    if (Gate2) Gate2->Configure(Relay,NAME_None);
    // The hazard is visible via signage, not via an art VFX asset.
    AExposureZone* Cold=GetWorld()->SpawnActor<AExposureZone>(FVector(3980,0,130),FRotator::ZeroRotator);
    if (Cold) {Cold->WarmthDrainPerSecond=17.f;Cold->BoxExtent=FVector(400,620,200);}
    Sign(TEXT("WARNING: FREEZING EXPOSURE"),FVector(3710,0,230),FColor::Red);
    if (AGreyPickup* Warm=SpawnPiece<AGreyPickup>(PickupClass,FVector(4340,-360,55)))
        Warm->Configure(EPickupKind::Warmth,NAME_None,55.f);
    SpawnPiece<AGreyExit>(ExitClass,FVector(4870,0,125));
}
`````

## File: ThresholdRun/Source/ThresholdRun/World/GreyWorldBuilder.h

`````cpp
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Engine/World.h"
#include "GreyWorldBuilder.generated.h"
class ASwitchMechanism;
class APressurePlate;
class ALogicRelay;
class AGreyGate;
class AGreyPickup;
class AGreyExit;
UCLASS(Blueprintable)
class THRESHOLDRUN_API AGreyWorldBuilder : public AActor
{
    GENERATED_BODY()
public:
    AGreyWorldBuilder();
    UPROPERTY(EditDefaultsOnly,BlueprintReadOnly,Category="Blueprint Extensions") TSubclassOf<ASwitchMechanism> SwitchClass;
    UPROPERTY(EditDefaultsOnly,BlueprintReadOnly,Category="Blueprint Extensions") TSubclassOf<APressurePlate> PlateClass;
    UPROPERTY(EditDefaultsOnly,BlueprintReadOnly,Category="Blueprint Extensions") TSubclassOf<AGreyGate> GateClass;
    UPROPERTY(EditDefaultsOnly,BlueprintReadOnly,Category="Blueprint Extensions") TSubclassOf<AGreyPickup> PickupClass;
    UPROPERTY(EditDefaultsOnly,BlueprintReadOnly,Category="Blueprint Extensions") TSubclassOf<AGreyExit> ExitClass;
    UPROPERTY(EditAnywhere,BlueprintReadWrite,Category="Generation") bool bAllowOptionalBranch = true;
    virtual void BeginPlay() override;
private:
    AActor* Block(const FVector& Center,const FVector& Dimensions);
    void Sign(const FString& Text,const FVector& Location,const FColor& Color = FColor::Cyan);
    void MakeRoom(int32 Slot,bool bBranch);
    void MakeOptionalBranch();
    template<typename T> T* SpawnPiece(TSubclassOf<T> Class,const FVector& Location)
    {
        FActorSpawnParameters P;
        P.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;
        return GetWorld()->SpawnActor<T>(Class ? Class.Get() : T::StaticClass(),Location,FRotator::ZeroRotator,P);
    }
};
`````

## File: ThresholdRun/Source/ThresholdRun.Target.cs

`````csharp
using UnrealBuildTool;
using System.Collections.Generic;
public class ThresholdRunTarget : TargetRules
{
    public ThresholdRunTarget(TargetInfo Target) : base(Target)
    {
        Type = TargetType.Game;
        DefaultBuildSettings = BuildSettingsVersion.V5;
        IncludeOrderVersion = EngineIncludeOrderVersion.Latest;
        ExtraModuleNames.Add("ThresholdRun");
    }
}
`````

## File: ThresholdRun/Source/ThresholdRunEditor.Target.cs

`````csharp
using UnrealBuildTool;
using System.Collections.Generic;
public class ThresholdRunEditorTarget : TargetRules
{
    public ThresholdRunEditorTarget(TargetInfo Target) : base(Target)
    {
        Type = TargetType.Editor;
        DefaultBuildSettings = BuildSettingsVersion.V5;
        IncludeOrderVersion = EngineIncludeOrderVersion.Latest;
        ExtraModuleNames.Add("ThresholdRun");
    }
}
`````

## File: ThresholdRun/ThresholdRun.uproject

`````json
{
  "FileVersion": 3,
  "EngineAssociation": "5.8",
  "Category": "Games",
  "Description": "Native C++ first-person survival puzzle roguelike greybox vertical slice",
  "Modules": [{"Name":"ThresholdRun","Type":"Runtime","LoadingPhase":"Default"}],
  "Plugins": [{"Name":"EnhancedInput","Enabled":true}]
}
`````

## File: ThresholdRun/Tools/Build-Playtest.ps1

`````powershell
param(
    [string]$EngineRoot,
    [string]$ProjectRoot = (Split-Path -Parent $PSScriptRoot)
)
. "$PSScriptRoot\Windows.Common.ps1"
if ([Environment]::OSVersion.Platform -ne [PlatformID]::Win32NT) {
    throw 'An Unreal Win64 game must be built on Windows with UE 5.8 and its C++ toolchain.'
}

# Reuse installed engines; never download or install an engine silently.
if (!$EngineRoot) {
    $candidates = @()
    if ($env:UE_ROOT) { $candidates += $env:UE_ROOT }
    $engineKey = 'HKLM:\SOFTWARE\EpicGames\Unreal Engine\5.8'
    if (Test-Path -LiteralPath $engineKey) {
        $candidates += (Get-ItemProperty -LiteralPath $engineKey -Name InstalledDirectory).InstalledDirectory
    }
    $candidates += Join-Path $env:ProgramFiles 'Epic Games\UE_5.8'
    foreach ($candidate in $candidates) {
        $versionFile = Join-Path $candidate 'Engine\Build\Build.version'
        if (!(Test-Path -LiteralPath $versionFile)) { continue }
        $version = Get-Content -Raw -LiteralPath $versionFile | ConvertFrom-Json
        if ($version.MajorVersion -eq 5 -and $version.MinorVersion -eq 8) {
            $EngineRoot = $candidate
            break
        }
    }
}
if (!$EngineRoot) {
    throw 'UE 5.8 was not found. Install it, or run Build-Playtest.cmd -EngineRoot "D:\Epic Games\UE_5.8".'
}
$paths = Get-ThresholdPaths $EngineRoot $ProjectRoot
$builds = Join-Path $paths.Project 'Builds'
$output = Join-Path $builds 'Windows'
if (Test-Path -LiteralPath $output) {
    # Preserve all previous artifacts rather than deleting or merging them.
    $previous = Join-Path $builds ('Windows-previous-' + [Guid]::NewGuid().ToString('N'))
    Move-Item -LiteralPath $output -Destination $previous
    Write-Host "Previous archive preserved at $previous"
}

Write-Host 'Preparing editor and greybox map...'
& "$PSScriptRoot\Prepare-Windows.ps1" -EngineRoot $paths.Engine -ProjectRoot $paths.Project
Write-Host 'Building the Windows Development game...'
& "$PSScriptRoot\Package-Windows.ps1" -EngineRoot $paths.Engine -ProjectRoot $paths.Project
Write-Host 'Checking the packaged game and save persistence across restarts...'
& "$PSScriptRoot\Run-SmokeTests.ps1" -EngineRoot $paths.Engine -ProjectRoot $paths.Project -Packaged

$launchers = @(Get-ChildItem -LiteralPath $output -Filter ThresholdRun.exe -Recurse |
    Where-Object { $_.FullName.Substring($output.Length) -notmatch '[\\/]Binaries[\\/]' })
if ($launchers.Count -ne 1) { throw "Could not identify the packaged bootstrap executable under $output" }
$playtestZip = Join-Path $builds ('ThresholdRun-Windows-Playtest-' + [Guid]::NewGuid().ToString('N') + '.zip')
# Include cooked content, DLLs, prerequisites and the bootstrap, not just the EXE.
# Use .NET ZIP support instead of Compress-Archive's single-file size limitation.
Add-Type -AssemblyName System.IO.Compression.FileSystem
[IO.Compression.ZipFile]::CreateFromDirectory($output, $playtestZip)
Write-Host "PASS: Windows playtest packaged and automated checks completed."
Write-Host "Playable launcher: $($launchers[0].FullName)"
Write-Host "Share the complete playtest archive: $playtestZip"
try {
    Start-Process -FilePath explorer.exe -ArgumentList ('/select,"' + $launchers[0].FullName + '"')
} catch {
    Write-Host "Build succeeded. Open the playable launcher shown above manually."
}
`````

## File: ThresholdRun/Tools/Create-GreyboxMap.py

`````python
"""Run ONLY in Unreal Editor via Prepare-Windows.ps1; never overwrites a map."""
import json
import os
from pathlib import Path

import unreal


def prepare():
    receipt = Path(os.environ['THRESHOLD_MAP_RECEIPT'])
    content = Path(unreal.Paths.convert_relative_path_to_full(unreal.Paths.project_content_dir()))
    map_file = content / 'Maps' / 'L_Greybox.umap'
    level = unreal.get_editor_subsystem(unreal.LevelEditorSubsystem)
    actors = unreal.get_editor_subsystem(unreal.EditorActorSubsystem)
    asset_path = '/Game/Maps/L_Greybox'
    created = not map_file.exists()
    if created:
        if not level.new_level(asset_path):
            raise RuntimeError('Could not create /Game/Maps/L_Greybox')
        start = actors.spawn_actor_from_class(
            unreal.PlayerStart, unreal.Vector(0, 0, 160), unreal.Rotator(0, 0, 0))
        if not start:
            raise RuntimeError('PlayerStart spawn failed')
        world = unreal.get_editor_subsystem(unreal.UnrealEditorSubsystem).get_editor_world()
        game_mode = unreal.load_class(None, '/Script/ThresholdRun.GreyGameMode')
        if not game_mode:
            raise RuntimeError('Native GreyGameMode was not compiled')
        world.get_world_settings().set_editor_property('default_game_mode', game_mode)
        if not level.save_current_level():
            raise RuntimeError('L_Greybox save failed')
    elif not level.load_level(asset_path):
        raise RuntimeError('Existing L_Greybox could not be loaded; it was not overwritten')

    starts = [a for a in actors.get_all_level_actors() if isinstance(a, unreal.PlayerStart)]
    if len(starts) != 1:
        raise RuntimeError('Expected exactly one PlayerStart; existing map was not changed')
    position = starts[0].get_actor_location()
    rotation = starts[0].get_actor_rotation()
    if max(abs(position.x), abs(position.y), abs(position.z - 160), abs(rotation.yaw)) > 0.1:
        raise RuntimeError('PlayerStart must be (0,0,160), yaw 0; existing map was not changed')
    if not map_file.is_file():
        raise RuntimeError('Saved map file is missing')
    receipt.write_text(json.dumps({'status': 'prepared', 'created': created, 'map': asset_path}), encoding='utf-8')
    unreal.log('ThresholdRun: L_Greybox map preparation succeeded')


# A missing receipt fails the PowerShell runner even if the editor exits with 0.
try:
    prepare()
finally:
    unreal.SystemLibrary.quit_editor()
`````

## File: ThresholdRun/Tools/Package-Windows.ps1

`````powershell
param(
    [Parameter(Mandatory=$true)][string]$EngineRoot,
    [string]$ProjectRoot = (Split-Path -Parent $PSScriptRoot)
)
. "$PSScriptRoot\Windows.Common.ps1"
$paths = Get-ThresholdPaths $EngineRoot $ProjectRoot
$map = Join-Path $paths.Project 'Content\Maps\L_Greybox.umap'
if (!(Test-Path -LiteralPath $map)) { throw 'Missing L_Greybox.umap. Run Tools\Prepare-Windows.ps1 first.' }
$output = Join-Path $paths.Project 'Builds\Windows'
if (Test-Path -LiteralPath $output) {
    if (@(Get-ChildItem -LiteralPath $output -Force).Count -gt 0) {
        throw 'Builds\Windows is not empty. Move the previous archive aside before packaging; it will not be overwritten.'
    }
}
$uat = Join-Path $paths.Engine 'Engine\Build\BatchFiles\RunUAT.bat'
& $uat BuildCookRun "-project=$($paths.UProject)" -noP4 -platform=Win64 `
    -clientconfig=Development -build -cook '-map=/Game/Maps/L_Greybox' `
    -stage -pak -package -archive "-archivedirectory=$output" -prereqs -utf8output -unattended
if ($LASTEXITCODE -ne 0) { throw "Packaging failed with code $LASTEXITCODE" }
$executables = @(Get-ChildItem -LiteralPath $output -Filter ThresholdRun.exe -Recurse |
    Where-Object { $_.FullName -match '[\\/]Binaries[\\/]Win64[\\/]' })
if ($executables.Count -ne 1) { throw "Expected one packaged Development game executable under $output" }
Write-Host "Development Win64 archive: $output"
Write-Host "Game binary: $($executables[0].FullName)"
`````

## File: ThresholdRun/Tools/Prepare-Windows.ps1

`````powershell
param(
    [Parameter(Mandatory=$true)][string]$EngineRoot,
    [string]$ProjectRoot = (Split-Path -Parent $PSScriptRoot),
    [ValidateRange(1,32)][int]$MaxParallelActions = 8
)
. "$PSScriptRoot\Windows.Common.ps1"
$paths = Get-ThresholdPaths $EngineRoot $ProjectRoot
$build = Join-Path $paths.Engine 'Engine\Build\BatchFiles\Build.bat'
& $build ThresholdRunEditor Win64 Development "-Project=$($paths.UProject)" `
    -WaitMutex -NoHotReloadFromIDE "-MaxParallelActions=$MaxParallelActions"
if ($LASTEXITCODE -ne 0) { throw "Editor build failed with code $LASTEXITCODE" }

$runRoot = Join-Path $paths.Project ('Saved\Automation\Map-' + [Guid]::NewGuid().ToString('N'))
New-Item -ItemType Directory -Path $runRoot | Out-Null
$receipt = Join-Path $runRoot 'prepared.json'
$previousReceipt = [Environment]::GetEnvironmentVariable('THRESHOLD_MAP_RECEIPT', 'Process')
try {
    $env:THRESHOLD_MAP_RECEIPT = $receipt
    Invoke-ThresholdProcess -FilePath (Join-Path $paths.Engine 'Engine\Binaries\Win64\UnrealEditor-Cmd.exe') `
        -TimeoutSeconds 600 -Arguments @(
            $paths.UProject, '-unattended', '-nop4', '-nosplash', '-NullRHI',
            '-EnablePlugins=PythonScriptPlugin',
            "-ExecutePythonScript=$PSScriptRoot\Create-GreyboxMap.py",
            "-abslog=$runRoot\MapPreparation.log"
        )
} finally {
    [Environment]::SetEnvironmentVariable('THRESHOLD_MAP_RECEIPT', $previousReceipt, 'Process')
}
if (!(Test-Path -LiteralPath $receipt)) { throw "Map preparation failed. Inspect $runRoot\MapPreparation.log" }
$prepared = Get-Content -Raw -LiteralPath $receipt | ConvertFrom-Json
if ($prepared.status -ne 'prepared') { throw 'Unexpected map preparation result' }
Write-Host "Editor compiled; L_Greybox saved/validated. Evidence: $runRoot"
`````

## File: ThresholdRun/Tools/Run-SmokeTests.ps1

`````powershell
param(
    [Parameter(Mandatory=$true)][string]$EngineRoot,
    [string]$ProjectRoot = (Split-Path -Parent $PSScriptRoot),
    [switch]$Packaged,
    [ValidateRange(60,3600)][int]$TimeoutSeconds = 180,
    [switch]$Render
)
. "$PSScriptRoot\Windows.Common.ps1"
$paths = Get-ThresholdPaths $EngineRoot $ProjectRoot
$runRoot = Join-Path $paths.Project ('Saved\Automation\Smoke-' + [Guid]::NewGuid().ToString('N'))
$sandbox = Join-Path $runRoot 'User'
New-Item -ItemType Directory -Path $sandbox -Force | Out-Null
if ($Packaged) {
    # Launch the real binary directly so timeout/exit monitoring covers the game,
    # rather than the short-lived prerequisite/bootstrap executable.
    $executables = @(Get-ChildItem -LiteralPath (Join-Path $paths.Project 'Builds\Windows') `
        -Filter ThresholdRun.exe -Recurse | Where-Object { $_.FullName -match '[\\/]Binaries[\\/]Win64[\\/]' })
    if ($executables.Count -ne 1) { throw 'Expected one packaged game binary. Package first.' }
    $executable = $executables[0].FullName
    $prefix = @('/Game/Maps/L_Greybox')
} else {
    $executable = Join-Path $paths.Engine 'Engine\Binaries\Win64\UnrealEditor-Cmd.exe'
    $prefix = @($paths.UProject, '/Game/Maps/L_Greybox', '-game')
}

# A fresh process exercises TC1-3; a second process reuses only the isolated disk
# save and proves GameInstance::Init reloads permanent progress on application start.
foreach ($phase in @('Fresh', 'Restart')) {
    $report = Join-Path $runRoot $phase
    $receipt = Join-Path $sandbox "$phase.ok"
    $arguments = $prefix + @(
        '-unattended', '-nosplash', '-nosound', '-nop4', '-SaveToUserDir',
        "-UserDir=$sandbox", "-ThresholdSmokeRoot=$sandbox",
        '-ExecCmds=Automation RunTests ThresholdRun',
        '-TestExit=Automation Test Queue Empty', "-ReportExportPath=$report",
        "-abslog=$runRoot\$phase.log"
    )
    if (!$Render) { $arguments += '-NullRHI' }
    if ($phase -eq 'Restart') { $arguments += '-ThresholdSmokeRestart' }
    Write-Host "Running $phase smoke phase; log: $runRoot\$phase.log"
    Invoke-ThresholdProcess -FilePath $executable -Arguments $arguments -TimeoutSeconds $TimeoutSeconds
    Assert-ThresholdReport $report
    if (!(Test-Path -LiteralPath $receipt) -or (Get-Content -Raw -LiteralPath $receipt).Trim() -ne 'PASS') {
        throw "Missing success receipt for $phase; inspect the current report and log."
    }
}
Write-Host "PASS: TC1 spawn/vitals; TC2 switch/gate <=2s; TC3 oxygen death/reload/save; separate-process persistence."
Write-Host "Reports, logs and isolated save retained at $runRoot"
`````

## File: ThresholdRun/Tools/Test-VerificationHelpers.ps1

`````powershell
# No Unreal Engine or Pester required. Tests report acceptance/rejection only.
. "$PSScriptRoot/Windows.Common.ps1"
$testDirectory = Join-Path ([IO.Path]::GetTempPath()) ('ThresholdReportTest-' + [Guid]::NewGuid().ToString('N'))
New-Item -ItemType Directory -Path $testDirectory | Out-Null
$checks = 0
function Write-ReportFixture {
    param([string]$State = 'Success', [string]$Name = 'ThresholdRun.Smoke.GameLoop',
        [int]$Succeeded = 1, [int]$Failed = 0, [int]$NotRun = 0, [int]$InProcess = 0,
        [switch]$Empty)
    $testEntries = @(@{ fullTestPath = $Name; state = $State })
    if ($Empty) { $testEntries = @() }
    @{ tests = $testEntries; succeeded = $Succeeded; succeededWithWarnings = 0;
       failed = $Failed; notRun = $NotRun; inProcess = $InProcess } |
        ConvertTo-Json -Depth 5 | Set-Content -LiteralPath (Join-Path $testDirectory 'index.json')
}
function Assert-Rejected {
    $rejected = $false
    try { Assert-ThresholdReport $testDirectory } catch { $rejected = $true }
    if (!$rejected) { throw 'Invalid/missing automation result was accepted' }
}
try {
    Write-ReportFixture
    Assert-ThresholdReport $testDirectory
    $checks++
    Write-ReportFixture -Empty -Succeeded 0
    Assert-Rejected
    $checks++
    Write-ReportFixture -State 'Fail' -Succeeded 0 -Failed 1
    Assert-Rejected
    $checks++
    Write-ReportFixture -State 'NotRun' -Succeeded 0 -NotRun 1
    Assert-Rejected
    $checks++
    Write-ReportFixture -State 'InProcess' -Succeeded 0 -InProcess 1
    Assert-Rejected
    $checks++
    Write-ReportFixture -Name 'Unrelated.Test'
    Assert-Rejected
    $checks++
    Remove-Item -LiteralPath (Join-Path $testDirectory 'index.json')
    Assert-Rejected
    $checks++
    '{invalid json' | Set-Content -LiteralPath (Join-Path $testDirectory 'index.json')
    Assert-Rejected
    $checks++
    Write-Host "PASS: $checks automation report validation checks (these are not gameplay tests)."
} finally {
    Remove-Item -LiteralPath $testDirectory -Recurse
}
`````

## File: ThresholdRun/Tools/Windows.Common.ps1

`````powershell
Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

function Get-ThresholdPaths {
    param([string]$EngineRoot, [string]$ProjectRoot)
    if ([Environment]::OSVersion.Platform -ne [PlatformID]::Win32NT) {
        throw 'These build/run commands require Windows with Unreal Engine 5.8.'
    }
    $engine = (Resolve-Path -LiteralPath $EngineRoot).Path
    $project = (Resolve-Path -LiteralPath $ProjectRoot).Path
    $uproject = Join-Path $project 'ThresholdRun.uproject'
    if (!(Test-Path -LiteralPath $uproject)) { throw "Missing $uproject" }
    $version = Get-Content -Raw -LiteralPath (Join-Path $engine 'Engine\Build\Build.version') | ConvertFrom-Json
    if ($version.MajorVersion -ne 5 -or $version.MinorVersion -ne 8) {
        throw 'This starter targets UE 5.8. Use that engine or explicitly port the project separately.'
    }
    return @{ Engine = $engine; Project = $project; UProject = $uproject }
}

function Invoke-ThresholdProcess {
    param([string]$FilePath, [string[]]$Arguments, [int]$TimeoutSeconds = 180)
    # Windows CRT quoting: preserve spaces, quotes and trailing backslashes.
    $quoted = foreach ($argument in $Arguments) {
        '"' + [regex]::Replace([regex]::Replace($argument, '(\\*)"', '$1$1\"'), '(\\+)$', '$1$1') + '"'
    }
    $process = Start-Process -FilePath $FilePath -ArgumentList ($quoted -join ' ') -PassThru
    if (!$process.WaitForExit($TimeoutSeconds * 1000)) {
        # Stop only this invocation and its children; never stop unrelated editors.
        & taskkill.exe /PID $process.Id /T /F | Out-Null
        throw "Timed out after $TimeoutSeconds seconds: $FilePath"
    }
    $process.Refresh()
    if ($process.ExitCode -ne 0) { throw "Process exited with code $($process.ExitCode): $FilePath" }
}

function Assert-ThresholdReport {
    param([string]$ReportDirectory)
    $index = Join-Path $ReportDirectory 'index.json'
    if (!(Test-Path -LiteralPath $index)) { throw "Missing automation report: $index" }
    $report = Get-Content -Raw -LiteralPath $index | ConvertFrom-Json
    $tests = @($report.tests)
    if ($tests.Count -ne 1 -or $tests[0].fullTestPath -ne 'ThresholdRun.Smoke.GameLoop' `
        -or $tests[0].state -ne 'Success' -or $report.failed -ne 0 `
        -or $report.notRun -ne 0 -or $report.inProcess -ne 0 `
        -or ($report.succeeded + $report.succeededWithWarnings) -ne 1) {
        throw "Smoke test failed, was skipped, or was not discovered: $index"
    }
}
`````

## File: ThresholdRun/Tools/validate_source.py

`````python
"""Static sanity checks. This cannot replace UnrealBuildTool / UHT / Editor Play tests."""
import json
import re
from pathlib import Path

ROOT = Path(__file__).resolve().parents[1]
SRC = ROOT / 'Source' / 'ThresholdRun'
assert json.loads((ROOT / 'ThresholdRun.uproject').read_text())['Modules'][0]['Name'] == 'ThresholdRun'
assert 'EnhancedInput' in (SRC / 'ThresholdRun.Build.cs').read_text()
assert (ROOT / 'Config' / 'DefaultInput.ini').is_file()
files = list(SRC.rglob('*.h')) + list(SRC.rglob('*.cpp'))
assert len(files) >= 25
for p in files:
    raw = p.read_text()
    if p.suffix == '.h' and p.name != 'ThresholdRun.Build.cs':
        if '.generated.h' in raw:
            assert raw.count('.generated.h') == 1, p
            includes = re.findall(r'^#include\s+"([^"]+)"', raw, re.M)
            assert includes[-1].endswith('.generated.h'), p
    # Only simple delimiter counts; comments/strings can create false positives.
    for a,b in [('(',')'),('{','}'),('[',']')]:
        assert raw.count(a) == raw.count(b), (p,a,raw.count(a),raw.count(b))
    for inc in re.findall(r'^#include\s+"([^"]+)"', raw, re.M):
        if inc.startswith(('Player/','UI/','Interaction/','Survival/','Puzzles/','Progression/','World/')):
            assert (SRC/inc).exists(), (p,inc)
needed = [
 'Player/GreyPlayer.cpp','Survival/SurvivalVitals.cpp','Puzzles/GreyGate.cpp',
 'Puzzles/PressurePlate.cpp','Puzzles/LogicRelay.cpp',
 'World/GreyWorldBuilder.cpp','Progression/GreyGameInstance.cpp'
]
for n in needed: assert (SRC/n).is_file()
builder=(SRC/'World/GreyWorldBuilder.cpp').read_text()
for token in ['Gate1->Configure(First','Gate2->Configure(Relay','Relay->Configure(Inputs,true)',
              'EPickupKind::Shortcut','EPickupKind::Blueprint','EPickupKind::Warmth',
              'FRandomStream','MakeOptionalBranch','APointLight']:
    assert token in builder, token
player=(SRC/'Player/GreyPlayer.cpp').read_text()
for token in ['EKeys::W','EKeys::E','EKeys::MouseX','LineTraceSingleByChannel',
              'Execute_CanInteract','Execute_Interact']:
    assert token in player, token
print(f'OK: {len(files)} C++ source files; valid .uproject; local includes, headers, FSM wiring, save and input structural checks.')
print('NOTE: UE5 compiler/UHT and runtime collision/input tests NOT executed.')
`````
