# Runtime evaluation: Winlator WB64 for Age of Empires III: Definitive Edition

## Decision

Use **Winlator WB64 as the first runtime candidate**, not as a dependency to embed immediately. First validate the user's own legally obtained Age of Empires III: Definitive Edition installation inside an existing WB64 installation on the REDMAGIC 8S Pro. Only after that succeeds should this repository add a launcher shell or attempt a fork.

Upstream project: https://github.com/winebox64/winlator-wb64

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
2. Install an official WB64 release and complete first-run setup.
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
