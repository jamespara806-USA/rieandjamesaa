# Threshold Run Windows playtest builder

Download `ThresholdRun_Windows_Playtest_Builder.zip` in this folder. On its GitHub file page, use **Download raw file**.

This archive contains the game source and Windows build launcher. It does not contain a compiled executable.

1. Extract the entire ZIP on a Windows PC with Unreal Engine 5.8 and its supported Visual Studio C++ toolchain / Windows SDK.
2. Open the extracted `ThresholdRun` folder.
3. Double-click `Build-Playtest.cmd`.
4. After compilation, packaging and smoke tests pass, Explorer selects `ThresholdRun.exe`. The `Builds` folder also contains a ZIP of the complete playable build for sharing.

See `START_HERE.md` inside the archive for details. Windows compilation and gameplay were not executed on the Linux cloud host. The ZIP integrity, source structure and script syntax were checked.


## Complete Claude handoff

[Download ThresholdRun_Claude_Handoff.zip](ThresholdRun_Claude_Handoff.zip?raw=true).

This includes every current project file, a detailed CLAUDE_HANDOFF.md, a ready-to-copy CLAUDE_PROMPT.txt, one combined CLAUDE_CONTEXT.md text document, BUILD_STATUS.json and SHA256 file manifests. All original starter files remain unchanged.

The handoff does not contain a compiled EXE, binary map or cooked content. Windows/Unreal compilation and runtime tests remain required. The owner has UE 5.8 on a Windows PC; build with Build-Playtest.cmd in the extracted ThresholdRun folder.

For a receiving chat that does not unpack ZIP files, use [CLAUDE_CONTEXT.md](CLAUDE_CONTEXT.md?raw=true). The shorter [CLAUDE_HANDOFF.md](CLAUDE_HANDOFF.md) explains architecture, exact class names, persistence, build steps and the boundary between executed checks and pending Unreal validation.
