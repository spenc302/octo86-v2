# Age of Empires III — Android

A device-first Android launcher and compatibility profile for running a legally owned copy of **Age of Empires III: Complete Collection** on the REDMAGIC 8S Pro (China variant), initially targeting the Snapdragon 8 Gen 2 / Adreno 740 / 16 GB RAM configuration.

> This repository does not contain the game, proprietary game assets, executables, or redistributable Windows files. Users must provide their own legally obtained game installation.

## Project direction

Build one installable Android app with a focused AoE3 launcher/UI and an integrated Windows-compatibility runtime based on Winlator WB64. The intended user experience is one APK installation; large runtime components may be unpacked or provisioned on first launch. The game installation remains user-supplied.

### Runtime investigation baseline

- Android host application / launcher
- Wine for Windows API compatibility
- Box86 for 32-bit x86 game components (confirm actual executable and dependency architecture before locking this in)
- Box64 only where 64-bit host/runtime components require it
- DXVK for supported Direct3D 9/10/11 paths translated to Vulkan
- Mesa Turnip Vulkan driver for the Adreno GPU, subject to device/driver compatibility
- Container/profile configuration, input mapping, audio and filesystem integration

**Important:** Box86 is not a Windows compatibility layer and DXVK is not bundled automatically by Box86. Wine, the correct 32-bit ARM userspace, Box86, Vulkan and DXVK must be configured as a compatible whole. AoE3's exact graphics API usage and 32-bit dependency chain must be verified experimentally.

## Target device

- REDMAGIC 8S Pro, China variant
- Qualcomm Snapdragon 8 Gen 2
- Adreno 740
- 16 GB RAM
- Android / REDMAGIC OS version to be recorded during device validation

## Milestones

1. **Runtime selection:** inspect a maintained Winlator/WineBox64 source and identify a reproducible, license-compliant runtime baseline.
2. **Game compatibility profile:** document the required Wine architecture, Box86/Box64 setup, DXVK version, Vulkan driver, display mode and environment settings.
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

Winlator WB64 has been selected as the runtime foundation for a single-install AoE3 app. The packaging target is one APK with runtime assets provisioned or unpacked on first launch if bundling everything would make the APK impractically large. The upstream Android app source has an Apache-2.0 license; the WB64 source mirror has an MIT top-level license, and each bundled dependency still needs its own license review. No Android APK has been built and no AoE3 compatibility has yet been demonstrated. See [runtime evaluation](docs/runtime-evaluation.md), [single-APK packaging](docs/single-apk-packaging.md), and the [device validation checklist](docs/device-validation.md). No GitHub Actions build has been run.
