# Octo86 UI shell integration

## Current source

The first game-focused home screen is authored at:

`integration/winlator-app-overlay/app/src/main/java/com/winlator/Aoe3HomeFragment.java`

It is intentionally separate from the upstream Winlator submodule for now. The upstream Android app is a Git submodule owned by `brunodev85/winlator-app`; this repository can pin its commit but cannot commit modifications inside that upstream repository. The shell source is therefore staged in this repository until an Octo86-owned fork of `winlator-app` is available.

## First-screen behavior

The fragment provides:

- Octo86 game-library header and an Age of Empires III: Definitive Edition profile card
- Game-folder selection entry point
- Existing Winlator container-management entry point
- Graphics settings entry point
- Controller/input settings entry point
- Explicit not-installed state; no false claim that the game is already imported or launchable

It uses Material Components already present in the upstream app dependencies. It does not create a second Wine/Box64 runtime.

## Host wiring required

The host `MainActivity` must implement `Aoe3HomeFragment.HostActions`:

- `onChooseGameFolder()`: launch Android's Storage Access Framework directory picker, persist the returned URI permission, then hand the selected tree to an import service that copies the game into a chosen container. Merely retaining the URI is not a complete import.
- `onOpenContainers()`: navigate to the existing `ContainersFragment`.
- `onOpenGraphicsSettings()`: navigate to the existing `SettingsFragment` (or a game-specific graphics profile screen once the per-game profile model exists).
- `onOpenInputSettings()`: navigate to the existing input-controls screen or the selected container's controls editor.

The default startup navigation in `MainActivity.onCreate()` must be changed to show `Aoe3HomeFragment` after first-run permissions and RootFS installation are handled. Existing deep links for input editing and file-manager navigation must continue to work.

## Next implementation stages

1. Create an Octo86-owned fork of `brunodev85/winlator-app` and point the `app` submodule in `upstream/winlator/.gitmodules` at that fork, pinned to a known commit. This is required before the actual app source can be changed safely.
2. Wire the home fragment into MainActivity and add navigation back to the original Winlator containers, shortcuts, settings and controls.
3. Implement game-folder import into a selected container with progress, cancellation, free-space checks and safe handling of Android document-provider URIs.
4. Detect the actual AoE3 DE executable and architecture from the imported files; do not hardcode a guessed executable path.
5. Add a dedicated per-game profile for Box64, Wine, DXVK/graphics driver, resolution, audio and controller settings. Keep unverified compatibility settings clearly marked as experimental.
6. Enable Launch only after import and executable validation succeed; create or update the Winlator shortcut using its existing shortcut model and launcher path.
7. Only then run the manually triggered Actions build and test the APK on the target phone.

No build should be run before these stages are sufficiently complete and the user explicitly asks to build.
