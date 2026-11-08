# DeviceDeck roadmap

## Phase 0 — Repository foundation

- [x] Create public umbrella repository and public platform forks.
- [x] Attach upstream remotes and organize existing Edifier research.
- [x] Record target-device matrix and evidence policy.
- [x] Pin upstream revisions and retain each base's license and history.
- [ ] Add CI metadata and a versioned device-profile format after the first profile is understood.

## Phase 1 — Inventory and protocol discovery

1. Decompile the supplied realme Link and Baseus Intelligent APK splits locally; identify package/build versions, model databases, classic/BLE transports, packet framing, capability discovery, and command mappings. This established that EP10 Pro/MF1 use RFCOMM/SPP, while Bowie 30 uses BLE and server-supplied GATT endpoints. The first macOS EP10 Pro backend now reads battery/noise and performs readback-confirmed Off/ANC/Transparency writes; it still needs hardware verification. Bowie 30 BLE, MF1 feature mapping, Blade 2 telemetry, and Android/Windows paths remain open. See [`research/baseus-intelligent-findings.md`](../research/baseus-intelligent-findings.md).
2. Extract device catalogs from all reviewed apps and open projects (QuickBuds, HeyBuds, Edifier Connect/ConneX, Baseus Intelligent, mEDIFIER, EncoDeck, and relevant platform clients). Maintain provenance per model and per feature.
3. Compare catalog/UI evidence with runtime packet evidence. Static strings and model rows are leads, not proof of working hardware support.
4. For Baseus Blade 2, first establish its BLE profile and whether it has any BLE control channel; power-bank telemetry and charging safety are treated separately.

## Phase 2 — Reliable protocol core

- Define capability, state, command, response, and error models.
- Add protocol codecs as vendor/model modules, with golden packet fixtures and provenance notes.
- Add read-before/write/read-after behavior, timeouts, reconnect handling, and capability-gated UI.
- Add a native BLE/GATT transport to Android and Windows/Linux before exposing Edifier or Baseus controls there. The inherited Android and desktop apps currently speak RFCOMM for their realme/OPPO link; the Android Edifier codec alone does not connect to BLE devices. Keep platform transport code separate from the vendor codec/profile.
- Keep destructive operations (reset, firmware update, arbitrary raw writes) out of ordinary controls until independently verified and guarded.

## Phase 3 — Initial device support

Suggested order:
1. realme Air 7 / Air 7 Pro on Android and Windows, using the QuickBuds foundation.
2. Edifier W820NB Plus on Android, Windows, and macOS, using the supplied Edifier reverse-engineering report and the platform app foundations.
3. realme Air 7 / Air 7 Pro on macOS.
4. Baseus EP10 Pro, Bowie 30, Bowie MF1, then Blade 2 based on protocol evidence and hardware access.

### Current implementation checkpoint (2026-11-08)

- macOS: W820NB Plus BLE discovery, raw D8 capability query, battery read, ANC/transparency, and a decoded D8 capability report are in [DeviceDeck-macOS #7](https://github.com/Basil-AS/DeviceDeck-macOS/pull/7). [DeviceDeck-macOS 0.7.0-alpha.4](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.4) adds capability-gated game mode (0x08 query, 0x09 write followed by readback), tests, a DMG, and source archive. No physical headset has verified connection or command behavior. EQ, prompts and gestures remain outstanding. The DMG is ad-hoc signed and not notarized.
- Android/Windows/Linux: DeviceDeck branding and protocol foundation are in [DeviceDeck-Android-Windows #2](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/2); versioned signing/release pipeline is in [#3](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/3). Alpha.2 artifacts are published: [Android APK/AAB](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.2) and [Windows ZIP/Linux AppImage](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.2), each with SHA256SUMS; the Android APK signature and package were verified by tag CI. This release adds the Air 7 Pro spatial-silent warning to the desktop known-issues screen and regression coverage. Android/desktop realme Air 7/Air 7 Pro protocol profiles remain inherited from QuickBuds and need fresh DeviceDeck hardware verification.
- Android: the isolated Edifier frame codec and tests are merged in [DeviceDeck-Android-Windows #9](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/9). Shared Android BLE/GATT transport, notification setup, serialized writes, bounded discovery and W820NB Plus/Baseus endpoint profiles are merged in [#10](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/10). [#12](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/12) adds W820NB Plus battery and ANC controls with readback in the same More devices hub as Baseus Bowie 30. Android unit tests/lint, APK/AAB release build, and shared Windows/Linux checks passed. This slice remains hardware-unverified; capability-gated EQ, game, prompts, gestures and remaining controls are still open.
- Windows/Linux: the inherited app's realme device link is RFCOMM. PR #13 adds a shared BLE/GATT accessory hub to the existing DeviceDeck desktop client, with W820NB Plus and Bowie 30 profiles. It reads battery/noise state and performs mode writes with readback; Android and desktop CI passed. The new profiles remain unverified on physical hardware, and EP10 Pro, MF1, Blade 2 and wider catalog support remain open.
- macOS EP10 Pro SPP and Bowie 30 GATT slices are implemented but hardware-unverified. [DeviceDeck-macOS #19](https://github.com/Basil-AS/DeviceDeck-macOS/pull/19) is merged; it adds the short “EP10 Pro” catalog alias without modifying imported upstream resources. Android Bowie 30 is in the same DeviceDeck hub and BLE session ([#11](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/11)); Windows/Linux also has the shared accessory hub ([#13](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/13)). Android [#14](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/14) is merged: Blade 2/Blade 2-140 dashboard telemetry (including 140 cell metrics), display theme/screensaver/text and timers are in the same hub; Android test/lint/release builds and Windows/Linux tests/archives passed. Physical-device round-trip remains pending. Windows/macOS Blade 2 UI/session and Android/Windows EP10 Pro and MF1 controls remain open. macOS MF1 battery-only SPP implementation is under review in [#20](https://github.com/Basil-AS/DeviceDeck-macOS/pull/20).

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
