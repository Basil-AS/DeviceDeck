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

- macOS: [DeviceDeck 0.7.0-alpha.18](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.18), Android: [0.1.0-alpha.16](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.16), and Windows/Linux: [0.1.0-alpha.16](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.16) are published from the unified DeviceDeck apps. The newest feature is the W820NB Plus prompt-tone switch, gated by D8 B17 bit 1 and read/written with F9/FA plus readback. Tagged CI passed; Android APK/AAB, Windows ZIP, Linux AppImage, macOS DMG, and source ZIP were downloaded, checksum-checked, and archive/image-verified. This is static APK-derived behavior and remains unverified on physical headphones. The macOS DMG is ad-hoc signed and not notarized.

- macOS: [DeviceDeck 0.7.0-alpha.17](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.17) is published with a CI-built DMG and source archive. It adds W820NB Plus touch assignment mapping in the existing DeviceDeck panel. Android/Windows/Linux alpha.15 is also published; APK/AAB and ZIP/AppImage tag workflows, checksums and release jobs passed. The controls use D8 profile 18, F0 reads, and F1 writes with readback; mapping evidence is static Edifier Connect analysis and needs physical-device validation.

- macOS: [DeviceDeck 0.7.0-alpha.16](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.16) and Android/Windows/Linux: [DeviceDeck 0.1.0-alpha.14](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.14) / [desktop-v0.1.0-alpha.14](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.14) add the capability-gated W820NB Plus touch-sensitivity query and setting in the existing unified device panels. D8 gates FD/FE; writes are checked with a fresh FD read. Release CI passed and artifact hashes/integrity were verified. Hardware round-trip remains outstanding; controls are shown only when reported supported and the response is valid.

- macOS: [DeviceDeck 0.7.0-alpha.15](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.15) adds capability-gated W820NB Plus Bluetooth name query and rename inside the existing DeviceDeck device panel. C9 reads, CA writes, and C9 readback follow D8 capability and UTF-8 byte-limit reporting. Release CI passed; the downloaded DMG and source ZIP passed integrity checks. Hardware behavior remains unverified; the DMG is ad-hoc signed and not notarized.
- Android: [APK/AAB 0.1.0-alpha.13](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.13) and Windows/Linux: [ZIP/AppImage 0.1.0-alpha.13](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.13) include the same W820NB Plus Bluetooth name controls in the unified apps. Tag CI passed; downloaded APK/AAB/ZIP/AppImage checksums matched release manifests, archives were valid, and the AppImage is x86-64 ELF. Hardware behavior remains unverified.

- macOS: [DeviceDeck 0.7.0-alpha.14](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.14) adds capability-gated Edifier W820NB Plus no-audio auto power-off in the unified device panel. CI and the release DMG/source-package audit passed. The DMG is ad-hoc signed and not notarized; headset read/write/readback still needs hardware validation.
- Android: [DeviceDeck 0.1.0-alpha.12 APK/AAB](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.12) is published. Windows/Linux: [DeviceDeck 0.1.0-alpha.12 ZIP/AppImage](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.12) is published. Tag workflows passed, SHA256SUMS matched the downloaded files, APK/AAB/ZIP archive checks passed, and the AppImage is an x86-64 ELF executable. Android signature verification passed in CI. Hardware read/write/readback remains outstanding.
- macOS: [DeviceDeck 0.7.0-alpha.13](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.13) includes Blade 2 C1/C2 port controls and the shared Edifier, realme, and Baseus profiles. CI passed; the downloaded source ZIP passed `unzip -t` and the DMG passed `hdiutil verify`. Hardware behavior remains unverified; the DMG is ad-hoc signed, not notarized.
- Android: [APK/AAB 0.1.0-alpha.11](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.11) includes EP10 Pro gesture actions and BA8E/BA8F cycle-mode choices, alongside Blade 2 C1/C2 port controls and the W820NB Plus and other Baseus profiles. Release CI passed; downloaded APK/AAB hashes match `SHA256SUMS`, and the APK passed `unzip -t`.
- Windows/Linux: [ZIP/AppImage 0.1.0-alpha.11](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.11) includes the same DeviceDeck profiles and controls. Windows ZIP and AppImage checksums were recorded; the ZIP passed `unzip -t`, and the AppImage is an x86-64 ELF executable. AppImage packaging uses a checksum-verified, MIT-licensed pinned AppRun runtime to avoid upstream asset download failures on hosted runners.
- Remaining target gaps include physical-device validation on all platforms, Edifier prompts/gestures, Baseus MF1 gestures, Blade 2 port-control hardware validation and reset mapping, additional Air 7/Air 7 Pro controls, and support expansion from the reviewed device catalogs. Keep all brands and device profiles inside each platform's unified DeviceDeck app.

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
