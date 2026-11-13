# Single-APK packaging plan

## User-facing target

The deliverable should be one installable Android APK. The user should not need to install Winlator as a separate app.

The APK will contain the AoE3-focused launcher and the Android-side runtime integration. On first launch, it can provision or unpack large runtime payloads into the app's private storage, then create the initial Wine/container environment. The user supplies their own legally obtained Age of Empires III game installation.

This is still a single-app installation experience even if first launch needs additional setup time or storage. It is not the same as a completely offline, self-contained APK.

## Two packaging modes

### Preferred: one APK + first-launch runtime provisioning

- APK contains launcher, integration code, required native Android libraries, manifests and runtime bootstrap logic.
- Runtime payloads are either included as compressed assets or fetched from a project-controlled release location after verifying hashes and versions.
- First launch unpacks/provisions the payload into app-private storage.
- Show download size, storage requirement, progress, retry and integrity errors clearly.
- Do not fetch proprietary Windows game files or Microsoft game content.
- Pin exact runtime component versions and checksums; do not silently follow moving upstream releases.

This keeps the app installable as one APK while avoiding an enormous base package. It does require internet access on first setup if payloads are hosted rather than embedded.

### Alternative: fully bundled/offline APK

- Bundle every redistributable runtime component in the APK as compressed assets.
- Unpack to app-private storage on first launch.
- No runtime download is needed, but the APK and installed storage footprint will be much larger.
- This mode must wait until we measure the actual payload sizes and finish the license audit.

## Integration approach

1. Use the maintained Winlator WB64 Android app source as the reference/base, rather than trying to launch another installed app or inventing a private cross-app API.
2. Replace the general-purpose home screen with a focused AoE3 launcher only after confirming the upstream runtime can start the target game.
3. Keep the container/runtime management needed by Wine, Box86/Box64, graphics drivers, audio and input. Do not strip components just because the first UI is game-specific.
4. Add an AoE3 profile for the verified game executable, display mode, renderer, controls and logs.
5. Keep the original game installation outside the APK and ask the user to select/import their own files.

## Licensing gate

The WB64 source mirror's top-level LICENSE is MIT. The separate current Winlator Android source repository's top-level LICENSE is Apache-2.0. These are not a blanket licence for every component shipped by the application.

Before distributing an APK, create a third-party notices inventory that records, for every bundled component:
- exact project and pinned version/commit;
- source URL;
- applicable licence and required notices;
- whether source publication or offer is required;
- build modifications and attribution;
- whether binary redistribution is permitted under the relevant licence.

The inventory must include the relevant Wine, Box86/Box64, Mesa/Turnip, DXVK and other runtime packages, plus any root filesystem packages. Do not assume a component is redistributable solely because upstream WB64 bundles it.

## Size and storage

Measure both compressed download size and installed/unpacked size before choosing the final mode. Android app packaging supports large assets through different delivery mechanisms, but the project is currently targeting a directly installable APK, not Play Store delivery. The actual size and device storage budget must be measured rather than guessed.

## Current decision

Target: **one APK install, first-launch runtime provisioning**.

Do not start a build or GitHub Actions workflow yet. First inspect the upstream source layout and dependency packaging, pin a known runtime baseline, and confirm the AoE3 executable/renderer requirements.

## Upstream source inspection (2026-11-13)

The WB64 source mirror at `winebox64/winlator` states that the only source it provides is v7.1 and directs readers to Bruno's main Winlator repository. Its advertised v8 source folder is not a source tree: it contains only a README and an image. The WB64 mirror therefore cannot be treated as a complete, buildable v8 source checkout.

The separate upstream repository `brunodev85/winlator-app` is a real Android/Gradle source tree (default branch `main`). Its README identifies it as the Android application for running Windows x86_64 apps with Wine and Box86/Box64, and its top-level licence is **LGPL-2.1**. It is a viable app-base candidate, but it is not proof of WB64-specific patch parity.

### What the source inspection confirms

- The app is a single Android application module (`:app`) using Gradle, Android Gradle Plugin 8.4.2, compile SDK 35, min SDK 26, target SDK 28, and CMake 3.22.1 / NDK 24.
- It has native/JNI code and a substantial Java runtime-management layer: root filesystem installation, Wine setup, containers, graphics drivers, DX wrappers, shortcuts, display and input controls.
- Its current repository tree contains 70 packaged asset files totalling approximately **142.6 MiB** (sum of the GitHub-reported compressed file sizes). Examples include `rootfs.tzst` (~62.2 MiB), `direct3d.tzst` (~29.8 MiB), `container_pattern.tzst` (~7.1 MiB), and Box64, DXVK, Turnip and other driver/component archives.
- The app's `RootFSInstaller` extracts `rootfs.tzst` from Android assets into app-private storage on first setup; this is already the basic one-APK / first-launch provisioning pattern we wanted. The project does not need a separate Winlator installation at runtime if we integrate this application code and its licensed assets correctly.
- The inspected tree contains Box64 assets but no path named Box86. The README's general Box86/Box64 credit is not sufficient to establish how 32-bit Windows executables are supported in this exact build. We must inspect the Wine/Box64 build configuration and verify the AoE3 executable's PE architecture before choosing a runtime path.
- The repository's Android source is LGPL-2.1, and the runtime archives have their own component licences. The 142.6 MiB asset total is a useful packaging baseline, not a confirmed final APK size or permission to redistribute every archive.

### Integration decision

- Use `brunodev85/winlator-app` as the current Android app-base candidate, subject to reconciling the WB64-specific changes that may be missing.
- Preserve the upstream LGPL-2.1 notices and source obligations. Create a separate component-by-component notices manifest for Wine, Box64, Mesa/Turnip, DXVK, audio, rootfs packages, and every other redistributed archive.
- Pin exact commits for the app and all native/runtime submodules before importing or building. The upstream `brunodev85/winlator` parent repository currently pins `app` at `a030f552f452158a2db64fdb32b490fa19c0b48d`, `gladio` at `116c0d14dedbea3bd057f98f1db138bb1efe225e`, and `vortek` at `b1730c5def9b575672e671aee11d79ae7adc63d1`. The app repository's current `main` is newer (`2c33741d50c5daab0521a098be5c969f63b9e887`); use the parent-pinned app revision as the reproducible baseline unless we deliberately choose and document a newer compatible set. The pinned app tree was checked and contains the same rootfs asset, CMake project, and no Box86-named path.
- Keep the existing rootfs extraction/provisioning mechanism where possible. Do not copy the entire upstream binary payload into this repository until each archive's provenance and redistribution terms are checked.
- Before designing the AoE3 launcher shortcut, inspect the target game's executable architecture, required DirectX version, dependencies, and renderer. In particular, determine whether the game is 32-bit and how this upstream runtime's Wine/Box64 configuration supports it; do not assume Box86 is present.
- Do not strip the general container, display, audio, driver or input infrastructure until a working AoE3 launch path proves which parts are required.
- No GitHub Actions workflow or build has been run as part of this inspection.


## Runtime architecture findings (2026-11-13)

The parent-pinned app revision was inspected directly at `a030f552f452158a2db64fdb32b490fa19c0b48d`.

- `:app` is configured as an Android application with namespace/application ID `com.winlator`, compile SDK 35, min SDK 26, target SDK 28, Android Gradle Plugin 8.4.2, NDK 24.0.8215888 and CMake 3.22.1. The checked-in debug ABI filter is `arm64-v8a`, matching the target phone.
- `RootFSInstaller.installIfNeeded()` checks a version marker and extracts the bundled `rootfs.tzst` asset into app-private storage on first launch. It displays extraction progress and records the rootfs version after success.
- The rootfs installer also creates a versioned container template and resets affected container metadata when required.
- The Wine installer expects a 64-bit Wine binary and the rootfs layout includes both `x86_64-windows` and `i386-windows` Wine DLL directories; the container template has both `system32` and `syswow64`. This is evidence of a WoW64-style 32-bit Windows compatibility layout, even though there is no separately named Box86 asset. It is not yet proof that the specific AoE3 executable will launch correctly.
- The bundled Box64 archive is specifically `box64-0.4.4.tzst`. The graphics/runtime asset list also includes DXVK 1.10.3 and 2.4.1, D8VK 1.0, VKD3D 2.14.1, Turnip 26.2.0, Zink, VirGL, Vortek and Gladio packages.
- The largest checked-in archives are `rootfs.tzst` (~62.2 MiB), `direct3d.tzst` (~29.8 MiB), `container_pattern.tzst` (~7.1 MiB), `box64-0.4.4.tzst` (~4.3 MiB), `dxvk-2.4.1.tzst` (~4.1 MiB), and `rootfs_patches.tzst` (~4.0 MiB). The measured asset total remains about 142.6 MiB.
- The existing app has a broad container model (CPU configuration, graphics driver and DX wrapper selection, audio settings, Windows components, drive mappings, Box64 presets and shortcuts). We should retain that backend and only simplify the user-facing launcher after a known-good AoE3 profile works.

### Revised 32-bit compatibility assessment

The earlier concern that no Box86-named asset was present is narrowed, not fully resolved: this Wine build appears to use a 64-bit Wine installation with a WoW64 directory layout to support 32-bit Windows programs. We still need to inspect the Box64 launch component and the exact Wine build configuration to confirm how 32-bit PE processes are dispatched. AoE3: Complete Collection is expected to use a 32-bit Windows executable, so this remains a critical compatibility test.

### Next work before modifying app source

1. Inspect the exact guest-program launch command and Box64 environment setup, plus the pinned rootfs/Wine build metadata.
2. Identify the user's AoE3 executable and game version from a filename/PE-header report (no game files need to be uploaded).
3. Select a baseline container configuration for the game's DirectX path; do not assume DXVK is automatically the best renderer.
4. Only then fork/import the Android app source and replace the app identity, icon, and home screen. Keep the upstream runtime and notices intact.
5. No build or Actions workflow has been run.


## Wine replacement investigation (2026-11-13)

The user clarified that the Wine layer itself is the component to reconsider. The distinction below is important: several candidates are alternative Wine builds or Wine-derived compatibility stacks, not replacements for the Windows API implementation.

### Candidate assessment

1. **Kron4ek Wine-Builds — best first A/B candidate.** The project publishes Vanilla, Staging, Staging-TkG and Proton-derived builds, and explicitly documents an `amd64-wow64` build that runs 32-bit Windows applications without a separate 32-bit Linux library environment. This matches the current Box64-only architecture much more closely than a conventional 32-bit Wine build. However, its README also warns that ordinary Linux Wine dependencies are required and its published builds target desktop Linux/glibc, so the archive cannot simply be dropped into the Android rootfs. We would need to verify ABI/dependency compatibility and select a build specifically compiled for Box64/Android's glibc environment.
2. **Proton-GE — useful test variant, not a Wine replacement.** It is a Wine-derived compatibility tool with gaming patches. It is more complex than a plain Wine build and carries Steam-oriented assumptions; only consider it after a simpler WoW64 build has been tested.
3. **Hangover — interesting architecture, but still Wine-based.** Hangover runs x86/x86_64 Windows programs on AArch64 Wine through emulator DLLs. Its upstream documentation describes Box64 as an available emulator backend and its 32-bit dispatch through `wowbox64.dll`. This could reduce reliance on a complete x86 Wine process, but it is not a clean replacement for Wine and would require substantial integration work into the current Android app/rootfs. It is a research branch, not the first implementation choice.
4. **FEX by itself — not a Wine replacement.** FEX emulates x86 Linux programs; Windows API compatibility still needs Wine/Hangover. FEX upstream explicitly says Android is not its target platform, so do not select it as the direct Android runtime replacement.
5. **A completely different Windows API layer** would mean a major reimplementation or a native game port. There is no drop-in, maintained Android replacement that can simply take Wine's place for a conventional Windows PC game.

### Recommended approach

- Do not remove Wine from the app yet. Keep the runtime modular so the Wine payload can be swapped independently of Box64, graphics, input and container code.
- First test a **Box64-compatible x86_64 WoW64 Wine build** as an alternate Wine payload. Verify its ELF architecture, WoW64 support, glibc baseline, required shared libraries, Wine server, prefix initialization, and 32-bit Direct3D DLL loading.
- Compare the alternate against the existing Wine 10.10 payload with the same clean prefix and same game build. Capture Wine logs and test startup, menu, skirmish, audio, save/load and graphics separately.
- Only after that baseline should we test Staging/Proton patches or Hangover's emulator-DLL architecture.
- Do not mix Wine binaries from different builds in one prefix. Each Wine candidate should have its own isolated prefix and pinned version.
- No runtime was downloaded, no source or binary was copied into the project, and no Actions workflow/build was run during this investigation.


## Winlator@Frost / Glibc 7.1.3 assessment (2026-11-13)

The user suggested Winlator@Frost and its Glibc 7.1.3+ variant as a possible foundation.

### What it is

- Winlator@Frost is an Android Winlator fork, not a Wine replacement. It still uses Wine plus Box86/Box64.
- The Frost README describes a separate Glibc 7.1.3 mod/release alongside its regular APK. Therefore "Glibc 7.1.3" refers to a Frost runtime/distribution variant, not a Wine version number.
- Frost documents Turnip support for Snapdragon Adreno 6xx/7xx GPUs. The user's Snapdragon 8 Gen 2 / Adreno 740 is in that stated supported family.
- Frost documents custom Wine installation/import. This is valuable because it lets us distinguish the Android/container base from the selected Wine build.
- Frost's README warns that the Glibc variant has package-name/bench limitations and recommends fresh installation when switching variants; its containers are not necessarily portable between releases.
- The current Frost download page advertises a later 11.x release using Wine 10.10, while the Glibc 7.1.3 reference in the Frost source README is an older Glibc-mod release. Do not conflate the Glibc version label with Wine 7.1.3 or assume that the old mod's internals match the latest Frost release.

### Relevance to AoE3

Frost is a strong reference implementation for the Android-side runtime, graphics selection, component installation, controller/input handling, and Wine switching. It may be a better Android app base than importing the older upstream Winlator baseline unchanged.

However, Glibc itself does not implement Windows APIs and cannot replace Wine. A Glibc build can change the Linux userspace/runtime and possibly improve compatibility or performance for particular configurations, but the actual 32-bit Windows execution path still depends on the Wine build, WoW64 support, Box64 integration, and required libraries.

### Revised recommendation

1. Evaluate Frost's **Glibc 7.1.3 runtime as an Android runtime/base candidate**, not as the Wine replacement.
2. Inspect its exact Wine payload and custom-Wine install path; determine whether its shipped Wine is the same family/build as our current Wine 10.10 baseline.
3. Independently test a compatible WoW64 Wine payload through Frost's custom-Wine mechanism.
4. Compare PRoot and Glibc on the same device, same game files, same Wine version, same prefix configuration, and same DXVK/Turnip versions. Change one variable at a time.
5. If Frost's Glibc build is selected, port its relevant runtime integration and required glibc patches into our own app only after a component-by-component source and licence audit. Do not copy the whole APK or assume its runtime assets are freely redistributable.

No APK was installed, no binary was imported, and no Actions workflow/build was run during this assessment.


## Frost source audit (2026-11-13)

Inspected the accessible Frost source mirror at `LLazari/Winlator-Frost` (tree `c5217234802fcd6ba5f6f887825062beff7c3f7f`) and cross-checked its README against the upstream project references. The mirror identifies itself as a fork of `MrPhryaNikFrosty/Winlator-Frost`; its source README explicitly says the checked-in app source is only version 7.1 and is not updated with every newer release. Treat it as a useful implementation reference, not as source for the latest Frost 11.x APK.

### What the source confirms

- **Custom Wine is a real feature in the checked-in app source.** `WineUtils.extractWineFileForInstallAsync()` imports a user-selected XZ-compressed Wine archive into `/opt/installed-wine/preinstall/wine`. `findWineVersionAsync()` launches the candidate's Wine executable through the guest launcher, parses its reported version, and determines whether the Wine executable is x86 or x86_64. Installed Wine builds are then exposed through the container's Wine-version selector.
- **Wine is separated from the Android app/runtime at the filesystem level.** The default Wine is under `/opt/wine`; imported builds live under `/opt/installed-wine`. `WineInfo` chooses the executable for the default Wine's WoW64 or 32-bit mode, while imported builds use their own `wine64` or `wine` executable. This is a useful pattern for our swappable Wine-payload design.
- **There is an important limitation in this old source:** the WoW64 toggle is enabled only for the built-in/default Wine entry. Selecting an imported Wine build disables that toggle. Imported Wine is detected as x86 or x86_64, but the source does not demonstrate that an arbitrary imported x86_64 archive is a working WoW64 build. We must validate the actual payload and its 32-bit dispatch instead of assuming custom-Wine import solves that.
- **Wine prefixes are built around the selected Wine's DLL layout.** The source has separate x86_64 and i386 Windows DLL directories and copies DLLs into the container's system32/syswow64 layout. The prefix/template and DLL population logic would need to be kept consistent with whichever Wine build is selected.
- **Glibc is not implemented as a normal switch inside this source tree.** The checked-in `glibc_patches` directory contains a small set of Android System V shared-memory compatibility patches. The app source still includes and builds PRoot. The Glibc 7.1.3 runtime distribution is described separately in the README/releases; this repository does not provide a complete Glibc rootfs build recipe or an end-to-end Glibc APK packaging pipeline.
- **The source is an older Android build baseline:** app module uses compile SDK 30, min SDK 26, target SDK 28, NDK 22.1.7171670, and CMake 3.22.1. It bundles `imagefs.txz`, `container_pattern.tzst`, Box64 archives, graphics wrappers/drivers and optional components. The installer expands the image filesystem into app-private storage. This is useful architecture reference, but it is not a drop-in upgrade over our current SDK/NDK baseline.
- **The mirror's top-level MIT licence is not a complete redistribution clearance.** Its README credits separate upstream projects for Wine, Box86/Box64, Mesa/Turnip, DXVK, PRoot, Ubuntu rootfs and other components. Any binaries or archives we reuse need their own provenance, version and licence audit.

### Decision for octo86-v2

Do not replace our app base wholesale with the Frost 7.1 source. Instead, reuse the *design patterns* that directly solve our requirements:

1. Keep a dedicated Wine payload directory and a manifest describing version, architecture, WoW64 mode, source URL/provenance, hashes and licence.
2. Add Wine selection as a runtime configuration, but validate ELF architecture, required libraries, Wine server, PE 32-bit dispatch and prefix DLL layout before marking a payload usable.
3. Keep Wine prefixes isolated by Wine payload identity; never switch the Wine binary under an existing prefix without an explicit migration/recreation step.
4. Treat Frost Glibc 7.1.3 as a separately distributed runtime candidate. We still need its exact release asset/source recipe and dependency ABI before we can build or integrate it. The checked-in Frost source alone is insufficient to reproduce that Glibc variant.
5. Use the user's Adreno 740 / Snapdragon 8 Gen 2 as the target for a controlled PRoot-vs-Glibc comparison, but hold Wine, prefix, game files, DX wrapper and graphics-driver versions constant while changing only the runtime layer.

### Audit status

- Source inspection only; no Frost APK installed or executed.
- No Frost binaries or source files copied into octo86-v2.
- No app source code changed.
- No GitHub Actions workflow or build run.


## AoE3 Complete Collection compatibility target (2026-11-13)

This section is specifically about the original **Age of Empires III (2007) / Complete Collection**, not Age of Empires III: Definitive Edition.

### Verified game requirements

- Steam's legacy Age of Empires III (2007) support page lists Windows XP, DirectX 9.0c, and the original low-end GPU requirements. This is the legacy game, not the later Definitive Edition. Reference: https://help.steampowered.com/en/wizard/HelpWithGameTechnicalIssue?appid=105450
- PCGamingWiki's original-game technical summary identifies the Windows executable as 32-bit x86 and the Windows renderer as Direct3D 9.0c. It also identifies DirectSound audio and Bink video. Reference: https://www.pcgamingwiki.com/wiki/AoE3
- The Complete Collection includes the base game and both expansions. Its legacy Steam listing is App ID 105450. The game's multiplayer service was retired in 2024; campaign and skirmish remain playable according to the publisher's Steam announcement. Reference: https://store.steampowered.com/oldnews/?appgroupname=Age+of+Empires+III%3A+Complete+Collection&appids=105450&feed=steam_community_announcements
- Do not confuse this with **Age of Empires III: Definitive Edition**. DE is a separate 64-bit-era product requiring DirectX 11 and substantially newer Windows hardware. It is not the target for this port.

### Runtime implications

The original Complete Collection's known requirements make this primarily a **32-bit Windows compatibility problem**, not a DirectX 11/VKD3D problem:

1. The selected Wine payload must correctly run 32-bit x86 Windows PE applications on the ARM64 Android host. Prefer a validated x86_64 WoW64 Wine build with Box64-compatible dispatch; a Wine build being labelled x86_64 is not sufficient proof.
2. Start with the Direct3D 9 path. Test DXVK's D3D9 implementation with a compatible Turnip Vulkan driver as one candidate; retain WineD3D as the fallback comparison. Do not prioritize VKD3D, which targets Direct3D 12, or assume D8VK is relevant to this D3D9 game.
3. The game's DirectSound dependency means audio compatibility needs its own test. The existence of an Android audio backend does not by itself prove that the game's DirectSound calls are handled correctly.
4. Bink cutscenes should be tested separately from in-game rendering; a menu or skirmish test alone may not establish video playback compatibility.
5. Test base game and expansion executables separately if present. The exact executable names and PE headers must be read from the user's own installed copy before hard-coding launch targets or claiming the exact build is verified.

### Proposed first compatibility profile

- Host ABI: Android arm64-v8a / Snapdragon 8 Gen 2.
- Windows app: x86 (32-bit) AoE3 Complete Collection.
- Wine: validated x86_64 WoW64 build with a working Box64 dispatch path.
- Graphics: D3D9 through DXVK + Turnip as the first experiment; WineD3D as the controlled fallback.
- Audio: test the current audio backend with DirectSound.
- Prefix: fresh, isolated prefix per Wine payload; no cross-build prefix reuse.
- Scope: offline campaign and skirmish. Do not design around the retired legacy multiplayer service.

### Remaining device-specific verification

The sources establish the game's published architecture/API requirements, but they do not inspect the user's exact installed executable or prove that the current Android runtime launches it. Before implementation is locked, collect only non-game-content metadata from the installed copy: executable filename, PE machine type (expected x86 / IMAGE_FILE_MACHINE_I386), and file version. No proprietary game assets need to be uploaded. Then validate launch and graphics on-device when a testable APK/runtime is available.

No game files were downloaded or copied. No build or GitHub Actions workflow was run.


## Expanded game targets: AoE III original + Definitive Edition (2026-11-13)

The project now treats both editions as intended compatibility targets, with separate game profiles over a shared Android runtime.

### Target A — Age of Empires III (2007) / Complete Collection

- Windows app architecture: 32-bit x86.
- Graphics API: Direct3D 9.
- Initial graphics candidate: DXVK D3D9 + Turnip; WineD3D fallback.
- Runtime requirement: verified 32-bit PE dispatch through the selected ARM64-compatible Wine/Box64 arrangement.
- Preserve a separate prefix and game profile.

### Target B — Age of Empires III: Definitive Edition

Official Age of Empires support lists these minimum requirements: 64-bit Windows 10 (version 18362 or later), 64-bit processor/OS, 8 GB RAM, DirectX 11, 2 GB VRAM, and 42 GB storage. Recommended memory is 16 GB and recommended VRAM is 4 GB. Reference: https://support.ageofempires.com/hc/en-us/articles/360049678611-What-are-the-minimum-requirements-for-Age-of-Empires-III-Definitive-Edition

Steam lists the same core platform/API requirements and 42 GB storage. Reference: https://store.steampowered.com/app/933110/Age_of_Empires_III__Definitive_Edition/

Runtime implications:
- Target the x86_64 Windows executable path through Box64 + 64-bit Wine. Unlike the original game, this edition does not need the 32-bit WoW64 path as its primary execution mode.
- Direct3D 11 translation is required. Evaluate DXVK's D3D11 implementation with Turnip first; keep WineD3D as a diagnostic/fallback only if it exposes a usable D3D11 path. Do not use VKD3D as the default assumption: VKD3D primarily targets Direct3D 12.
- Expect materially higher GPU, memory, storage, and thermal pressure than the original edition. The phone's 16 GB system RAM is not equivalent to 16 GB of dedicated VRAM; Android, the container, Wine, Box64, graphics driver, and game all share physical memory.
- The official support page states that Android devices are not supported platforms. This is therefore an experimental compatibility project, not an officially supported mobile port.
- Keep the DE prefix, DLL overrides, graphics settings, and game configuration isolated from the original game's prefix.

### Shared application design

The Android APK should provide one shared container/runtime manager, but separate per-game profiles:

| Setting | Original Complete Collection | Definitive Edition |
|---|---|---|
| Windows app target | x86 | x86_64 |
| Wine path | WoW64-capable build | 64-bit Wine build |
| CPU translation | Box64 + verified 32-bit dispatch | Box64 x86_64 |
| Graphics API | D3D9 | D3D11 |
| First translation candidate | DXVK D3D9 | DXVK D3D11 |
| Prefix | Dedicated | Dedicated |
| Initial priority | Compatibility baseline | Higher-demand experimental target |

A shared runtime does not mean forcing both games into one Wine prefix or one set of DLL overrides. The launcher should identify each game profile and select its own Wine mode, prefix, DXVK architecture/DLL set, resolution, and performance defaults.

### Recommended implementation order

1. Stabilize the common Android container, Box64, graphics driver, audio and input layers.
2. Establish the original game's D3D9/32-bit baseline because it is the simpler compatibility target.
3. Add a distinct x86_64 + D3D11 profile for Definitive Edition.
4. Test each edition independently, then add a game selector and per-title settings to the UI.
5. Treat performance on the Snapdragon 8 Gen 2 / Adreno 740 as unverified until an on-device run; minimum PC requirements do not predict Android translation performance.

No game binaries or assets were downloaded. No APK build or GitHub Actions run was started.
