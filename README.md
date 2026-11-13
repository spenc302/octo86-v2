# Age of Empires III — Android

A device-first Android launcher and compatibility profile for running a legally owned copy of **Age of Empires III: Definitive Edition** on the REDMAGIC 8S Pro (China variant), initially targeting the Snapdragon 8 Gen 2 / Adreno 740 / 16 GB RAM configuration.

> This repository does not contain the game, proprietary game assets, executables, or redistributable Windows files. Users must provide their own legally obtained game installation.

## Project direction

Build one installable Android app with a focused AoE3 launcher/UI and an integrated Windows-compatibility runtime based on the current upstream Winlator 11.2 Android source and its pinned runtime components. The intended user experience is one APK installation; large runtime components may be unpacked or provisioned on first launch. The game installation remains user-supplied.

### Runtime investigation baseline

- Android host application / launcher
- Wine for Windows API compatibility
- Box64 for the expected x86_64 Windows game/runtime path, subject to PE-header verification
- Box86 / WoW64 only if the installed executable or a required component proves to be 32-bit
- DXVK for the game's Direct3D 11 path translated to Vulkan
- Mesa Turnip Vulkan driver for the Adreno GPU, subject to device/driver compatibility
- Container/profile configuration, input mapping, audio and filesystem integration

**Important:** Box64 is a CPU-translation layer, not a Windows compatibility layer. Wine supplies the Windows APIs; DXVK translates Direct3D 11 to Vulkan. If any required Windows component is 32-bit, the selected Wine build must provide a verified WoW64 path (and the runtime must support its dispatch requirements). The game's executable architecture, Direct3D 11 path, launch dependencies, and compatibility must be verified against the user's installed edition.

## Target device

- REDMAGIC 8S Pro, China variant
- Qualcomm Snapdragon 8 Gen 2
- Adreno 740
- 16 GB RAM
- Android / REDMAGIC OS version to be recorded during device validation

## Milestones

1. **Runtime selection:** pin the current upstream Winlator 11.2 app source, submodules and runtime components to reproducible commits.
2. **Game compatibility profile:** document the required Wine architecture, Box64 setup, DXVK version, Vulkan driver, display mode and environment settings.
3. **First launch test:** launch the user's own AoE3 installation in an existing runtime before developing a custom wrapper.
4. **Launcher shell:** create an Android UI for game path selection, profile management, launch/logging and settings.
5. **RTS controls:** add configurable touch controls for camera movement, selection, commands, zoom and keyboard shortcuts.
6. **Packaging:** determine what can legally and technically be bundled, and keep proprietary game data user-supplied.

## Non-goals for the first milestone

- Rewriting or decompiling the game engine
- Recreating the original game assets
- Shipping Microsoft game files or a preinstalled game
- Running GitHub Actions builds before the runtime and architecture are validated

## Current status

The current upstream Winlator 11.2 source is the selected Android runtime foundation candidate; the older WB64 source snapshot is retained only as a historical implementation reference. The inspected upstream app module is a real Gradle Android application with compile SDK 35, min SDK 26, NDK 24.0.8215888, a Wine 10.10 baseline, a Box64 guest launcher, and first-launch root-filesystem provisioning. The parent repository supplies versioned runtime components including DXVK and Turnip. Each component still needs its own licence/provenance review. No Android APK has been built and no Definitive Edition compatibility has yet been demonstrated. See [runtime evaluation](docs/runtime-evaluation.md), [single-APK packaging](docs/single-apk-packaging.md), and the [device validation checklist](docs/device-validation.md). No GitHub Actions build has been run.
