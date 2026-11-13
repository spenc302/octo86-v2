# Runtime evaluation: Winlator WB64

## Decision

Use **Winlator WB64 as the first runtime candidate**, not as a dependency to embed immediately. The first technical milestone is to test the user's own Age of Empires III installation inside an existing WB64 installation on the target phone. Only after that succeeds should this repository add a launcher shell or attempt a fork.

Upstream project: https://github.com/winebox64/winlator-wb64

## What upstream documents

The WB64 README describes an Android app for running Windows x86_64 applications with Wine and Box86/Box64. It lists Android 9+, Vulkan 1.1+, and Adreno 610+ as minimum requirements, while warning that results vary by device and game. It credits PRoot, Ubuntu Focal root filesystem, Wine, Box86/Box64, Mesa (including Turnip), DXVK, VKD3D, D8VK, and other components.

The upstream repository's top-level LICENSE is MIT. That applies to the repository's own code under its terms; it does **not** replace or override the separate licenses and notices for Wine, Box86/Box64, Mesa, DXVK, root filesystem packages, or any other bundled component. A redistribution/fork plan needs a component-by-component notice and license review.

## Relevance to AoE3 Complete Collection

- The intended first executable is the user's own installation's AoE3 game executable (likely the 32-bit `age3y.exe` target used by the existing decompilation research; verify the exact installed executable and architecture rather than assuming all Complete Collection executables match).
- Box86 is relevant to 32-bit x86 Linux userspace programs; Box64 is for 64-bit x86_64 userspace programs. Neither replaces Wine.
- The actual Windows execution path depends on the Wine build/prefix and the runtime's 32-bit compatibility setup. Confirm this in a real test; do not assume that the presence of Box86 alone proves a 32-bit Windows game will work.
- DXVK translates supported Direct3D 9/10/11 calls to Vulkan. It is a candidate only if the selected AoE3 renderer/API path is compatible. Do not assume it solves every legacy Direct3D path.
- The target phone's Snapdragon 8 Gen 2 / Adreno 740 is above WB64's published minimum GPU tier on paper. This is not proof that the phone's Android firmware, Vulkan driver, thermal limits, or this game will work.

## Test sequence

1. Record the exact Android/REDMAGIC OS version and free storage on the phone.
2. Install an official WB64 release and let its first-run setup finish.
3. Confirm that a normal container starts and that its graphics backend is available.
4. Use a legally obtained AoE3 Complete Collection installation supplied by the user. Do not put game executables, assets, installers, or proprietary redistributables in this repository.
5. Identify the exact game executable architecture and launch it from the container.
6. Test the title screen, a skirmish, audio, mouse/touch input, resolution changes, and a sustained session. Record logs and the selected Wine/Box86/Box64/graphics settings.
7. Only after a repeatable successful launch should we decide whether to fork WB64, create a separate lightweight launcher, or keep WB64 as a required runtime.

## Not yet verified

- AoE3 launch or gameplay on the REDMAGIC 8S Pro.
- The precise Complete Collection executable and dependency architecture on the user's installation.
- Whether the game's renderer works through the available DXVK/Vulkan path.
- Whether a custom launcher can safely and reliably control WB64 containers without invasive changes.
- Whether upstream runtime components can be redistributed together in a new APK under all applicable licenses.

## Build policy

No GitHub Actions workflow or build has been started. Do not spend Actions minutes on an APK until the on-device runtime test and integration choice are settled.
