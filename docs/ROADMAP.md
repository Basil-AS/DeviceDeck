# DeviceDeck roadmap

## Phase 0 — Repository foundation

- [x] Create public umbrella repository and public platform forks.
- [x] Attach upstream remotes and organize existing Edifier research.
- [x] Record target-device matrix and evidence policy.
- [x] Pin upstream revisions and retain each base's license and history.
- [ ] Add CI metadata and a versioned device-profile format after the first profile is understood.

## Phase 1 — Inventory and protocol discovery

1. Decompile the supplied realme Link and Baseus Intelligent APK splits locally; identify package/build versions, model databases, classic/BLE transports, packet framing, capability discovery, and command mappings. Baseus findings establish RFCOMM/SPP for EP10 Pro/MF1 and BLE for Bowie 30/Blade 2. DeviceDeck now has integrated support slices on all platforms for Bowie 30, Blade 2, EP10 Pro, and MF1 battery reading, all inside the same platform app. These slices remain unverified on physical hardware. See [`research/baseus-intelligent-findings.md`](../research/baseus-intelligent-findings.md).
2. Extract device catalogs from all reviewed apps and open projects (QuickBuds, HeyBuds, Edifier Connect/ConneX, Baseus Intelligent, mEDIFIER, EncoDeck, and relevant platform clients). Maintain provenance per model and per feature.
3. Compare catalog/UI evidence with runtime packet evidence. Static strings and model rows are leads, not proof of working hardware support.
4. For Baseus Blade 2, first establish its BLE profile and whether it has any BLE control channel; power-bank telemetry and charging safety are treated separately.

## Phase 2 — Reliable protocol core

- Define capability, state, command, response, and error models.
- Add protocol codecs as vendor/model modules, with golden packet fixtures and provenance notes.
- Add read-before/write/read-after behavior, timeouts, reconnect handling, and capability-gated UI.
- Native BLE/GATT transports and shared accessory hubs are now integrated on Android and Windows/Linux. Continue improving discovery stability and model-specific session behavior; keep platform transport separate from vendor codecs and profiles.
- Keep destructive operations (reset, firmware update, arbitrary raw writes) out of ordinary controls until independently verified and guarded.

## Phase 3 — Initial device support

Suggested order:
1. realme Air 7 / Air 7 Pro on Android and Windows, using the QuickBuds foundation.
2. Edifier W820NB Plus on Android, Windows, and macOS, using the supplied Edifier reverse-engineering report and the platform app foundations.
3. realme Air 7 / Air 7 Pro on macOS.
4. Baseus EP10 Pro, Bowie 30, Bowie MF1, then Blade 2 based on protocol evidence and hardware access.

### Current implementation checkpoint (2026-11-09)

- macOS: [DeviceDeck 0.7.0-alpha.10](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.10) is published with realme Air 7/Air 7 Pro profiles (ANC levels, preset EQ, game/spatial controls, wear detection, gesture maps, Air 7 multipoint and find-device), Edifier W820NB Plus D8-profile EQ selection with D5 readback, and Baseus profiles in the same app. The DMG and source archive passed CI; downloaded asset digests match GitHub and `hdiutil verify` accepts the DMG. Hardware behavior remains unverified; the DMG is ad-hoc signed, not notarized.
- Android: [APK/AAB 0.1.0-alpha.7](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.7) publishes profile-gated W820NB Plus EQ presets (D8 profile detection; D5 read; C4 write with D5 readback) in the same hub as Baseus EP10 Pro/MF1, Blade 2, and Bowie 30 controls. APK/AAB release CI is in progress.
- Windows/Linux: [ZIP/AppImage 0.1.0-alpha.7](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.7) adds matching W820NB Plus EQ controls alongside EP10 Pro SPP noise controls, MF1 battery reading, Blade 2 telemetry/display/timers, and Edifier/Bowie 30 in the shared accessory hub. Desktop release CI is in progress.
- Remaining target gaps include physical-device validation on all platforms, Edifier prompts/gestures, Baseus EP10 Pro/MF1 gestures, Blade 2 port enable/disable and reset mapping, additional Air 7/Air 7 Pro controls, and support expansion from the reviewed device catalogs. Keep every device profile inside each platform's unified DeviceDeck app.

Deliver each model in slices: identification and battery/status; primary modes; audio/EQ; gestures and multipoint; advanced settings; diagnostics.

## Phase 4 — Broader model coverage

- Build a deduplicated catalog of devices exposed by considered apps.
- Group models only when packet behavior and capability maps prove protocol compatibility.
- Prioritize models with publicly documented protocols or contributor hardware access.
- Publish per-model confidence and OS coverage; never infer full support from a shared marketing family name.

## Phase 5 — UX, assets, and releases

- Apply the DeviceDeck visual system consistently across platforms while respecting upstream UI architecture.
- Create original app icons and device illustrations, or use assets with documented redistribution rights.
- Add pairing/connect/recovery guidance, device-specific settings, offline operation, and useful error messages.
- Android/Windows/Linux and macOS alpha release pipelines have been exercised with version-matched tags and checksummed/public assets.
- macOS artifacts are ad-hoc signed and not notarized; Gatekeeper-clean distribution needs Developer ID/notarization credentials.
- Preserve the local Android signing key and properties in an owner-controlled encrypted backup; GitHub Actions secrets are already configured.
- Keep release channels per platform and publish only version-matched assets; mark unverified protocol builds as alpha.
- Package public builds only after hardware verification and license review; mark unverified builds as alpha.

## Validation gate for every device feature

A control is complete only after capability discovery, read, write, read-back, invalid-state handling, reconnect, and persistence have been checked on named hardware/firmware. User-provided hardware or trustworthy captured traces are needed for final verification.
