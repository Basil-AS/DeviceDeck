# DeviceDeck roadmap

## Phase 0 — Repository foundation

- [x] Create public umbrella repository and public platform forks.
- [x] Attach upstream remotes and organize existing Edifier research.
- [x] Record target-device matrix and evidence policy.
- [x] Pin upstream revisions and retain each base's license and history.
- [ ] Add CI metadata and a versioned device-profile format after the first profile is understood.

## Phase 1 — Inventory and protocol discovery

1. Decompile the supplied realme Link and Baseus Intelligent APK splits locally; identify package/build versions, model databases, BLE services/characteristics, packet framing, capability discovery, and command mappings.
2. Extract device catalogs from all reviewed apps and open projects (QuickBuds, HeyBuds, Edifier Connect/ConneX, Baseus Intelligent, mEDIFIER, EncoDeck, and relevant platform clients). Maintain provenance per model and per feature.
3. Compare catalog/UI evidence with runtime packet evidence. Static strings and model rows are leads, not proof of working hardware support.
4. For Baseus Blade 2, first establish its BLE profile and whether it has any BLE control channel; power-bank telemetry and charging safety are treated separately.

## Phase 2 — Reliable protocol core

- Define capability, state, command, response, and error models.
- Add protocol codecs as vendor/model modules, with golden packet fixtures and provenance notes.
- Add read-before/write/read-after behavior, timeouts, reconnect handling, and capability-gated UI.
- Keep destructive operations (reset, firmware update, arbitrary raw writes) out of ordinary controls until independently verified and guarded.

## Phase 3 — Initial device support

Suggested order:
1. realme Air 7 / Air 7 Pro on Android and Windows, using the QuickBuds foundation.
2. Edifier W820NB Plus on Android, Windows, and macOS, using the supplied Edifier reverse-engineering report and the platform app foundations.
3. realme Air 7 / Air 7 Pro on macOS.
4. Baseus EP10 Pro, Bowie 30, Bowie MF1, then Blade 2 based on protocol evidence and hardware access.

### Current implementation checkpoint (2026-11-07)

- macOS: W820NB Plus BLE discovery, Edifier D8 capability query/raw-payload preservation, battery read, and ANC/transparency controls are merged in [DeviceDeck-macOS #2](https://github.com/Basil-AS/DeviceDeck-macOS/pull/2). DeviceDeck identity/release foundation is merged in [#3](https://github.com/Basil-AS/DeviceDeck-macOS/pull/3). The service UUID comes from the supplied mEDIFIER source archive; no physical headset has verified connection or command behavior. EQ, game mode, prompts, gestures, and device information are still outstanding.
- Android/Windows: foundation and GitFlow CI are in [DeviceDeck-Android-Windows #1](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/1). DeviceDeck identity, original launcher icon, own updater source, removal of upstream telemetry/report endpoints, and Windows/Linux branding are merged in [#2](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/2). Full Android and Windows/Linux CI is green. A persistent Android signing key and GitHub Actions secrets are configured locally/remotely. Tagged release publishing and the public-facing fork cleanup are in [PR #3](https://github.com/Basil-AS/DeviceDeck-Android-Windows/pull/3), awaiting CI; the signer has not yet been exercised by a tag release. Android/desktop realme Air 7/Air 7 Pro profiles are inherited from QuickBuds and need fresh DeviceDeck verification. No new protocol implementation has started there for Edifier or Baseus.
- Realme and Baseus requested models remain research/catalog-only in DeviceDeck until implementation and hardware evidence are recorded.

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
- Android/Windows: finish PR #3 CI, merge to develop, promote via a release PR to main, then exercise alpha tags and validate published assets/checksums/signing.
- macOS: tag-based packaging is configured, but ad-hoc signatures are not notarized; production Gatekeeper distribution needs Developer ID/notarization credentials.
- Keep release channels per platform and publish only version-matched assets; mark unverified protocol builds as alpha.
- Package public builds only after hardware verification and license review; mark unverified builds as alpha.

## Validation gate for every device feature

A control is complete only after capability discovery, read, write, read-back, invalid-state handling, reconnect, and persistence have been checked on named hardware/firmware. User-provided hardware or trustworthy captured traces are needed for final verification.
