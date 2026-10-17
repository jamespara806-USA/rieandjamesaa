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
