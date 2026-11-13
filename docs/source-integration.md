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

- It does not yet make the repository a standalone Android Studio project. The upstream project is nested under `upstream/winlator`; Gradle integration/root project wiring remains to be designed.
- It does not change Winlator's app identity, launcher UI, or container behavior.
- It does not add an AoE3 DE profile or game import workflow.
- It does not prove the game launches on the REDMAGIC 8S Pro.
- It does not create an APK or invoke GitHub Actions.

## Next engineering sequence

1. Inspect the upstream root Gradle/build files and app manifest at the pinned commit.
2. Decide whether the safest first delivery is a direct upstream-derived app fork or a wrapper module included in the same Android project. Avoid a second app process or private cross-app integration.
3. Keep the upstream package/application ID decision explicit; if changed, inspect authorities, deep links, file providers, and native/JNI assumptions.
4. Add the AoE3 DE profile only through the upstream container/shortcut mechanisms that actually exist in source.
5. Preserve input, audio, rootfs, Wine, Box64, DXVK, and Turnip systems until an on-device baseline works.
6. Perform the component-level licence/provenance audit before redistributing any runtime payload.
7. Build only after the user explicitly authorizes a build. No Actions minutes are to be used without that authorization.

## Licensing note

The upstream parent repository contains an LGPL-2.1 licence and declares `app`, `gladio`, and `vortek` as separate submodules. Their own source and binary components must be audited at the pinned revisions. The parent licence is not a blanket licence for every archive, driver, rootfs package, Wine build, or game dependency. Keep the upstream notices and provide corresponding source/offer information where required before distributing modified binaries.
