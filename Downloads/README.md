# Threshold Run Windows playtest builder

Download `ThresholdRun_Windows_Playtest_Builder.zip` in this folder. On its GitHub file page, use **Download raw file**.

This archive contains the game source and Windows build launcher. It does not contain a compiled executable.

1. Extract the entire ZIP on a Windows PC with Unreal Engine 5.8 and its supported Visual Studio C++ toolchain / Windows SDK.
2. Open the extracted `ThresholdRun` folder.
3. Double-click `Build-Playtest.cmd`.
4. After compilation, packaging and smoke tests pass, Explorer selects `ThresholdRun.exe`. The `Builds` folder also contains a ZIP of the complete playable build for sharing.

See `START_HERE.md` inside the archive for details. Windows compilation and gameplay were not executed on the Linux cloud host. The ZIP integrity, source structure and script syntax were checked.
