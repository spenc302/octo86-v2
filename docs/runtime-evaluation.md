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
