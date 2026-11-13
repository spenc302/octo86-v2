# Android build entry point

## Correct Gradle root

The upstream Winlator parent repository is a source/submodule aggregator. The actual Android Gradle project is the nested app repository:

`upstream/winlator/app/`

That repository contains its own:

- `settings.gradle` including `:app`
- top-level `build.gradle` declaring Android Gradle Plugin 8.4.2
- `gradle/wrapper/gradle-wrapper.properties` pinning Gradle 8.14.5
- `app/build.gradle` configuring namespace/application ID `com.winlator`, compile SDK 35, min SDK 26, NDK 24.0.8215888, and CMake 3.22.1
- native CMake entry point at `app/src/main/cpp/CMakeLists.txt`

The native CMake entry point adds the Winlator, Vortek, VirGL, MIDI, Adreno-tools, and Gladio renderer source directories within the app module. The app module's debug build is configured for `arm64-v8a`, matching the target phone's ABI.

## Clone and initialize source

From a machine with Git installed:

```sh
git clone --recurse-submodules https://github.com/spenc302/octo86-v2.git
cd octo86-v2
git checkout integrate/winlator-11.2
git submodule update --init --recursive
```

If the repository is already cloned, run the final submodule command before building.

## Build command

Run this from the repository root:

```sh
cd upstream/winlator/app
./gradlew :app:assembleDebug
```

On Windows, use `gradlew.bat :app:assembleDebug`.

Expected debug APK path:

```
upstream/winlator/app/app/build/outputs/apk/debug/app-debug.apk
```

This is a build instruction only. No build has been run, no APK has been produced, and the project has not yet been validated on the REDMAGIC 8S Pro. A successful Gradle build would establish packaging/build integrity, not prove that Age of Empires III: Definitive Edition launches or performs acceptably.

## Important limits

- The root of `octo86-v2` is not yet an Android Studio project; open `upstream/winlator/app` as the Android project after recursive submodules are initialized.
- Preserve the upstream `com.winlator` application ID for the first build. The manifest FileProvider authority and other package-sensitive references have not yet been audited for a rename.
- Do not add the game executable, game assets, or proprietary game redistributables to this repository.
- Do not run GitHub Actions for this project without explicit authorization. The local Gradle command above does not consume GitHub Actions minutes.
- The upstream app's third-party runtime payloads and licences still require a component-by-component redistribution review before any APK is shared.
