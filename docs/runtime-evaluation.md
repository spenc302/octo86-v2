# Runtime evaluation: Winlator 11.2 for Age of Empires III: Definitive Edition

## Decision

Use **current upstream Winlator 11.2 as the primary Android runtime candidate**, not the older WB64 source mirror. Validate the user's own legally obtained Definitive Edition installation in the upstream app before adding a custom launcher. Keep the app source and runtime components pinned; do not copy the entire runtime blindly or start a build until the on-device validation path is clear.

Upstream project: https://github.com/brunodev85/winlator

The older WB64 source audit below is historical and is superseded by the 2026-11-13 upstream Winlator 11.2 source audit at the end of this document.

## Target game and official requirements

Primary target: **Age of Empires III: Definitive Edition** (Steam App ID 933110).

Official minimum requirements specify 64-bit Windows 10, 8 GB RAM, DirectX 11, 2 GB VRAM and 42 GB storage. Recommended requirements specify 16 GB RAM and 4 GB VRAM. The publisher states that Android and Linux are not supported platforms. These are PC requirements and do not predict performance under Android translation.

Official requirements: https://support.ageofempires.com/hc/en-us/articles/360049678611-What-are-the-minimum-requirements-for-Age-of-Empires-III-Definitive-Edition

## Executable discovery

The official support article identifies the executable paths:
- Steam: `AoE3DE_s.exe` in the AoE3DE installation folder.
- Microsoft Store: `AoE3DE.exe` under the package's Game folder.

Official support: https://support.ageofempires.com/hc/en-us/articles/360048536912-The-game-crashes-shortly-after-it-launches-or-when-trying-to-start-a-game

The 64-bit OS requirement is a strong signal for a 64-bit game environment, but it is not itself a PE-header inspection. Before hard-coding the Wine architecture, verify the actual installed executable's PE machine type and note any 32-bit helper processes or DLL dependencies. Do not request or upload proprietary game binaries; metadata from the user's own installation is sufficient.

## Runtime direction

- **Wine:** start with a current 64-bit Wine build that is compatible with the selected Box64 runtime. Confirm the actual Windows PE architecture before finalizing the mode.
- **CPU translation:** Box64 is the expected primary translator for x86_64 Linux guest components. If the game or a required component is 32-bit, the runtime must additionally provide a tested WoW64 / Box86-compatible dispatch path.
- **Graphics:** Direct3D 11 is the game's required API. Evaluate DXVK D3D11 with Turnip on Adreno 740 first. Keep a separate WineD3D or alternate-driver experiment for diagnosis; do not combine multiple changes in one test.
- **Glibc vs PRoot:** keep this as a controlled runtime comparison. Hold Wine, prefix, DXVK, Turnip, and game files constant; change only the Linux userspace/runtime layer.
- **Audio/video/input:** test these independently after the game process and renderer initialize. A successful title-screen launch does not establish complete compatibility.
- **Prefix:** dedicated clean prefix for Definitive Edition and a given Wine payload. Never reuse one prefix across different Wine builds without an explicit migration plan.

Winlator WB64 describes itself as an Android app for running Windows x86_64 applications with Wine and Box86/Box64. It credits PRoot, Ubuntu rootfs, Wine, Box86/Box64, Mesa/Turnip, DXVK, VKD3D and other components. Its upstream minimums include Android 9+, Vulkan 1.1+, and Adreno 610+, while noting that results vary by device and game: https://github.com/winebox64/winlator-wb64

## Target device

- REDMAGIC 8S Pro, China variant
- Snapdragon 8 Gen 2
- Adreno 740
- 16 GB shared system RAM (not dedicated VRAM)
- Android / REDMAGIC OS build to be recorded during validation

The phone's 16 GB RAM does not equal the game's recommended 4 GB dedicated VRAM. Memory pressure, thermal limits, driver compatibility, and translation overhead are major unknowns.

## Test sequence

1. Record Android/REDMAGIC OS version and available storage.
2. Install the official Winlator 11.2 release and complete first-run setup.
3. Confirm that a container starts, desktop output renders, Vulkan/Turnip is available, and logs can be captured.
4. Identify the user's own Definitive Edition installation and store edition (Steam or Microsoft Store).
5. Record executable filename, PE machine type, file version, and relevant non-sensitive dependency metadata; do not upload proprietary binaries or game assets.
6. Launch the game with a fresh dedicated prefix and the simplest viable configuration.
7. Test process startup, title screen, campaign/skirmish loading, D3D11 rendering, audio, cutscenes, input, save/load, and a sustained session.
8. Change one variable at a time: Wine build, PRoot/Glibc, DXVK version, Turnip version, Box64 preset, then game graphics settings.
9. Only after repeatable success should we decide whether to fork WB64, build a separate launcher, or keep WB64 as the runtime.

## Not yet verified

- Definitive Edition launch or gameplay on the REDMAGIC 8S Pro.
- Exact PE machine type and helper/dependency architecture for the installed edition.
- Whether the current Wine/Box64 combination initializes the game.
- Whether DXVK D3D11 + Turnip renders correctly and performs acceptably.
- Whether audio, Bink cutscenes, online services, and input work.
- Whether a custom launcher can reliably control WB64 containers.
- Whether every runtime component can be redistributed in one APK under its own licence.

## Build policy

No GitHub Actions workflow or APK build has been started. Do not spend Actions minutes until the on-device runtime test and integration choice are settled.


## WB64 source-level audit (2026-11-13)

Inspected the public `winebox64/winlator-wb64` source tree at commit `32d381d08dffa6db7a90ba0f1d3af234b4fd0a08`, including the Android Gradle configuration and the Wine / guest-launcher implementation.

Source references:
- https://github.com/winebox64/winlator-wb64/tree/main
- https://github.com/winebox64/winlator-wb64/blob/main/app/build.gradle
- https://github.com/winebox64/winlator-wb64/blob/main/app/src/main/java/com/winlator/core/WineInfo.java
- https://github.com/winebox64/winlator-wb64/blob/main/app/src/main/java/com/winlator/core/WineUtils.java
- https://github.com/winebox64/winlator-wb64/blob/main/app/src/main/java/com/winlator/xenvironment/components/GuestProgramLauncherComponent.java

### Confirmed source details

- The accessible source tree is an older **7.1** app baseline, not the latest advertised WB64 app release. Its Gradle file uses compile SDK 30, min SDK 26, target SDK 28, NDK 22.1.7171670, and CMake 3.22.1.
- Its built-in WineInfo baseline is Wine **9.2 x86_64**.
- The checked-in assets include Box64 0.2.8 / 0.2.9, Box86 0.3.2 / 0.3.7, DXVK 0.96 / 1.10.3 / 2.3.1, and Turnip 24.1.0. These are evidence of the source snapshot's age, not a recommendation to ship those versions.
- The guest launcher starts the guest command through Box64. In its default WoW64 mode it omits the separate Box86 environment and enables `BOX64_MMAP32=1`; the alternate non-WoW64 mode adds Box86 and a 32-bit PRoot loader. This makes the WoW64 setting a key compatibility variable, not a cosmetic preference.
- Wine selection and prefix construction are coupled: the default Wine has separate x86_64 and i386 Windows DLL directories, and the container template populates system32/syswow64 from those directories. Custom Wine builds are imported and detected, but the payload's actual WoW64 capability and matching prefix layout still require validation.
- The Android app source contains a broad container manager, X server, input controls, audio service, graphics wrapper selection and component installer. This is a substantial existing platform implementation; a lightweight game launcher can be layered over it after a working game launch is proven.

### Decision

Do **not** blindly fork this exact source snapshot and treat it as a current production base for Definitive Edition. Use it as an implementation reference and compatibility test candidate. Before choosing the source base, compare its old source baseline against a maintained source branch whose Android build configuration and runtime assets can be reproduced. For the game itself, start with a current compatible 64-bit Wine + Box64 + DXVK D3D11 + Turnip stack, then verify the installed executable's PE machine type.

The current `octo86-v2` repository still contains documentation only; no Android app source has been added yet. No proprietary game files were copied, and no build or Actions run was started.


## Current upstream Winlator 11.2 source audit (2026-11-13)

The current upstream repository is `brunodev85/winlator`. Its README identifies the app as an Android application for x86_64 Windows applications using Wine and Box86/Box64. The current 11.2 release is listed as the latest release, and the parent repository's main branch was updated on 2026-11-02 (commit `30aafbdb9eea47b92669201621e03cd2cf04540d`).

References:
- https://github.com/brunodev85/winlator
- https://github.com/brunodev85/winlator/releases
- https://github.com/brunodev85/winlator/commit/30aafbdb9eea47b92669201621e03cd2cf04540d

### Source inspection findings

- The parent repository pins the Android app submodule at `a030f552f452158a2db64fdb32b490fa19c0b48d`, with Gladio at `116c0d14dedbea3bd057f98f1db138bb1efe225e` and Vortek at `b1730c5def9b575672e671aee11d79ae7adc63d1`. These are the inspected baseline pins; re-check before importing.
- The app submodule is a real Android Gradle project. Its `app` module uses compile SDK 35, min SDK 26, target SDK 28, Android Gradle Plugin 8.4.2, NDK 24.0.8215888 and CMake 3.22.1. It identifies itself as version 11.2.
- The app's `WineInfo` class sets Wine 10.10 as the main Wine version.
- `GuestProgramLauncherComponent` launches guest commands through Box64, sets the guest rootfs environment, and provides Box64 logging/preset integration.
- `RootFSInstaller` provisions the root filesystem into app-private storage on first launch. This directly matches the desired one-app installation model: the user installs one APK, then the app initializes its runtime data.
- The app source contains dedicated Wine installation/version-detection logic, container/prefix creation, graphics-driver and DirectX-wrapper selection, and input/audio integration.
- The current parent release/source information advertises Winlator 11.2; the release notes include Box64 updates and newer Turnip integration. The parent source history's latest inspected commit updates the Box64 installable-component catalogue, removing Box64 0.3.3 while retaining 0.3.5, 0.3.7 and 0.4.5 entries.
- The inspected app asset tree includes DXVK 2.4.1 and Turnip 26.2.0 archives. These are source-tree observations, not proof that either version is optimal for AoE3 DE or that every archive may be redistributed by this project.

### Revised decision

**Use the current upstream Winlator 11.2 repository as the source foundation candidate. Do not use the old WB64 7.1 mirror as the main codebase.** WB64 and Frost remain useful reference implementations for runtime patterns, but the current upstream Winlator tree is more current and already has the required Android app architecture.

For the first AoE3 DE test profile, start with:
- 64-bit Wine 10.10 baseline.
- Box64 x86_64 guest execution.
- DXVK D3D11 as the initial graphics translation candidate.
- Turnip on Adreno 740 as the initial Vulkan driver candidate.
- A fresh, isolated AoE3 DE prefix.
- Low-resolution / conservative graphics settings for the first launch test; tune only after startup and rendering work.

These are test candidates, not confirmed compatible settings. The installed game executable's PE machine type and required helper components still need verification. The publisher's 64-bit Windows requirement is not a substitute for a PE-header inspection.

### Integration and licensing guardrails

- Keep the upstream app and submodule revisions pinned. Do not track floating `main` branches in a reproducible build.
- Preserve the app source's actual licence and notices; the parent repository's licence and the app submodule's licence must be checked independently. The licence of the Android app does not automatically grant redistribution rights for Wine, Box64, Mesa/Turnip, DXVK, rootfs packages or other archives.
- Before publishing an APK, produce a component manifest with exact version, source URL, checksum, licence, notices and source-offer obligations.
- Prefer a fork/submodule-based integration over manually copying selected Java files. The launcher should be a focused layer on top of the complete runtime, not a replacement for its container, rootfs, graphics, audio or input subsystems.
- Do not include the game, game executables, proprietary game assets or Microsoft redistributables. The user supplies their own game installation.

### Current validation status

- Source inspection only; no Winlator APK was installed or run in this session.
- No AoE3 DE executable was inspected.
- No Android app source was copied into `octo86-v2` yet.
- No APK was built and no GitHub Actions workflow was run.
