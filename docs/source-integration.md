# Upstream Winlator source integration

## Branch and pin

- Integration branch: `integrate/winlator-11.2`
- Upstream repository: https://github.com/brunodev85/winlator
- Pinned upstream commit: `30aafbdb9eea47b92669201621e03cd2cf04540d`
- Local submodule path: `upstream/winlator`
- Submodule URL is recorded in the root `.gitmodules`.

The upstream repository is included as a Git submodule rather than copied into this project. This preserves upstream history and its own app/native submodule references. Clone this project with recursive submodules enabled:

```sh
git clone --recurse-submodules https://github.com/spenc302/octo86-v2.git
git checkout integrate/winlator-11.2
```

If already cloned:

```sh
git submodule update --init --recursive
```

## What this accomplishes

- Pins the exact upstream Android runtime source without flattening its history.
- Retains upstream `app`, `gladio`, and `vortek` submodule pins through the upstream parent commit.
- Gives the AoE3 project a stable base to inspect and later adapt.
- Avoids copying proprietary game files or third-party runtime binaries into this repository.

## What it does not accomplish

- It does not make the root of this repository an Android Studio project. The actual Gradle root is nested at `upstream/winlator/app`, which is itself a complete Android project with a wrapper. See [upstream build entry point](upstream-build-entrypoint.md) for the exact command and expected output path.
- It does not change Winlator's app identity, launcher UI, or container behavior.
- It does not add an AoE3 DE profile or game import workflow.
- It does not prove the game launches on the REDMAGIC 8S Pro.
- It does not create an APK or invoke GitHub Actions.

## Next engineering sequence

1. Open `upstream/winlator/app` as the Android Studio project and validate the upstream debug build before making app changes. The exact build entry point is documented in [upstream-build-entrypoint.md](upstream-build-entrypoint.md).
2. Decide whether the safest first delivery is a direct upstream-derived app fork or a wrapper module included in the same Android project. Avoid a second app process or private cross-app integration.
3. Keep the upstream package/application ID decision explicit; if changed, inspect authorities, deep links, file providers, and native/JNI assumptions.
4. Add the AoE3 DE profile only through the upstream container/shortcut mechanisms that actually exist in source.
5. Preserve input, audio, rootfs, Wine, Box64, DXVK, and Turnip systems until an on-device baseline works.
6. Perform the component-level licence/provenance audit before redistributing any runtime payload.
7. Build only after the user explicitly authorizes a build. No Actions minutes are to be used without that authorization.

## Licensing note

The upstream parent repository contains an LGPL-2.1 licence and declares `app`, `gladio`, and `vortek` as separate submodules. Their own source and binary components must be audited at the pinned revisions. The parent licence is not a blanket licence for every archive, driver, rootfs package, Wine build, or game dependency. Keep the upstream notices and provide corresponding source/offer information where required before distributing modified binaries.


## Android project integration findings (pinned app submodule)

Inspected the app submodule at `a030f552f452158a2db64fdb32b490fa19c0b48d`.

- The upstream Gradle root is deliberately small: `settings.gradle` includes only `:app`. The app module is the actual application; it is not published as a reusable Android library.
- The app module uses namespace and application ID `com.winlator`, version name `11.2`, compile SDK 35, min SDK 26, target SDK 28, AGP 8.4.2, NDK 24.0.8215888, and CMake 3.22.1.
- The manifest launches `com.winlator.MainActivity`, registers the X server display activity and foreground service, and declares a FileProvider authority hard-coded as `com.winlator.FileProvider`.
- `MainActivity.onCreate()` selects the normal Containers/Shortcuts UI and calls `RootFSInstaller.installIfNeeded()` after permission handling. It also handles shortcuts, container file manager entry, and input-control editing.
- The existing app is therefore a tightly integrated runtime manager, not a drop-in SDK/library. A separate wrapper app that launches it through a second Android package would complicate container lifecycle, filesystem permissions, and user experience.

### Integration choice

For the first single-APK implementation, **fork/derive the upstream app module in-place and customize its existing MainActivity/navigation**, rather than creating a second application that attempts to control Winlator externally. Keep the upstream container/runtime classes intact.

Before changing `com.winlator` to an AoE3-specific application ID, inventory and update all package-sensitive references, including manifest component names, FileProvider authority, JNI/native library loading, intent actions, preferences, and any hard-coded filesystem paths. The first integration step should preserve the upstream application ID until a debug build can validate a deliberate package rename.

No Gradle files, manifest, Java source, native source, or assets were modified in this integration step. No build was run.
