# REDMAGIC 8S Pro validation checklist — AoE III: Definitive Edition

Target device: REDMAGIC 8S Pro China variant, Snapdragon 8 Gen 2, Adreno 740, 16 GB shared system RAM.

This is a checklist, not a claim that the game already works.

## Phone and runtime details

- Android version:
- REDMAGIC OS version/build:
- Available internal storage:
- WB64 release/version:
- Runtime mode (PRoot or Glibc):
- Container architecture:
- Wine version/build:
- Box64 version/preset:
- Vulkan driver/backend:
- Turnip version:
- DXVK version:
- Display resolution:

## Game installation metadata

- Store edition (Steam / Microsoft Store):
- Game version:
- Main executable (Steam: `AoE3DE_s.exe`; Microsoft Store: `AoE3DE.exe`):
- PE machine type:
- File version:
- Required helper architecture, if any:
- Installation size / available storage:

Do not upload the executable, proprietary game assets, account data, or credentials. The filename, PE machine type, file version, and sanitized log excerpts are enough for initial diagnosis.

## Runtime smoke test

- [ ] WB64 installs and completes first-run setup.
- [ ] Container can be created and started.
- [ ] Desktop/display output renders correctly.
- [ ] Vulkan and selected graphics driver initialize.
- [ ] Audio output works.
- [ ] Mouse, keyboard, and controller input can be configured.
- [ ] A simple non-game Windows application starts.
- [ ] Logs can be exported or copied for diagnosis.

## Definitive Edition test

- [ ] Exact game edition and executable identified.
- [ ] PE machine type confirmed from the installed copy.
- [ ] Fresh dedicated Wine prefix created.
- [ ] Game process starts.
- [ ] Main menu renders through D3D11.
- [ ] Campaign or skirmish loads.
- [ ] Camera, unit selection, and commands are usable.
- [ ] Audio and cutscenes work.
- [ ] Save/load works.
- [ ] No critical graphics corruption or repeatable crash.
- [ ] Sustained session completed; note heat, memory pressure, frame pacing, and stability.

## Test report

Record date, phone OS build, WB64 release, runtime mode, Wine/Box64/DXVK/Turnip versions, game version/executable metadata, settings, observed result, and relevant sanitized log excerpts. Never include passwords, account tokens, or unrelated personal data.
