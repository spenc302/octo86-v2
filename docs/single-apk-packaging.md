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
