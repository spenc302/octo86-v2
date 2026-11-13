# REDMAGIC 8S Pro validation checklist

Target recorded for this project: REDMAGIC 8S Pro China variant, Snapdragon 8 Gen 2, Adreno 740, 16 GB RAM.

This is a checklist, not a claim that the game already works.

## Phone details to record

- Android version:
- REDMAGIC OS version/build:
- Available internal storage:
- Vulkan driver/backend shown by WB64:
- WB64 release/version:
- Container architecture and Wine version:
- Box86/Box64 versions or presets:
- DXVK version and selected Direct3D backend:

## Runtime smoke test

- [ ] WB64 installs and completes first-run setup.
- [ ] A container can be created and started.
- [ ] Desktop/display output renders correctly.
- [ ] Audio output works.
- [ ] Input can be configured.
- [ ] A simple non-game Windows application starts.
- [ ] Logs can be exported or copied for diagnosis.

## AoE3 test

- [ ] Confirm the exact Complete Collection edition and executable.
- [ ] Confirm whether the selected executable is 32-bit x86.
- [ ] Confirm required game files are present from the user's own installation.
- [ ] Game process starts.
- [ ] Main menu renders.
- [ ] Skirmish loads.
- [ ] Camera movement, unit selection, and commands are usable.
- [ ] Audio and save/load work.
- [ ] No critical graphics corruption or repeatable crash.
- [ ] Sustained play test completed; note heat/performance and stability.

## Test report template

Record the date, phone OS build, WB64 release, container settings, exact executable, selected renderer, observed result, and relevant log excerpt. Avoid sharing account credentials or personal data in logs.
