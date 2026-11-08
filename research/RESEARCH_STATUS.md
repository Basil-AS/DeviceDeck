# Research status (2026-11-07)

## Completed

- Edifier Connect / ConneX static analysis from the supplied builds is archived in `edifier/EDIFIER_ANDROID_REVERSE_ENGINEERING.md` and `edifier/edifier_command_index.csv`.
- Supplied Baseus Intelligent 2.18.0 and realme Link 5.8.811.2814 split bundles were unpacked locally; their full bundle SHA-256 hashes are recorded in `APK_MANIFESTS.md`.
- Jadx produced local resource/source trees for the base APKs and reported recoverable errors (522 Baseus, 226 realme) after processing nearly all classes. Treat decompiled output as incomplete until key code paths are checked.
- Baseus Intelligent contains explicit profile/config entries for `Baseus Bass EP10 Pro`, `Bowie 30`, `Baseus Bowie MF1`, and `Blade 2`; examples appear in `DeviceResVersion1.java`, `DeviceManager.java`, and bundled gesture resource JSON. This proves model-specific app data exists, not that every advertised feature works over BLE.
- A focused Baseus report records model strings, resource categories, gesture function IDs, noise-mode UI evidence, and the current protocol boundary in [`baseus-intelligent-findings.md`](baseus-intelligent-findings.md). All four requested Baseus products remain research-only pending wire-level traces and hardware validation.
- Additional static tracing now documents Baseus's shared `BluetoothDataWriteManager` paths, `BAxx` to `789C` framing/CRC conversion, response parser, BLE/classic transport branch, and dynamically downloaded per-model resources. These mechanics are not a target-model command map; the report lists exactly what remains unknown.
- Targeted searches of the realme Link base APK source/resource output have not yet located the Air 7 / Air 7 Pro model names. Catalogs may be delivered through splits, native/assets data, or remote services; absence in this search is not proof of absent support.

## Next investigations

1. Inspect app split manifests and native libraries; trace the bundled Baseus BLE SDK write methods/UUID selection and identify BLE service discovery and model catalog/resource APIs. For Baseus, retrieve and version the exact target model resources and map each profile to supported control IDs.
2. Determine whether catalogs are bundled or remotely downloaded. If network-derived, inspect only lawful app behavior and record the exact response/source/version; do not publish credentials, tokens, or user identifiers.
3. Trace target model profiles and feature actions individually, then compare with QuickBuds/HeyBuds and the existing Edifier report.
4. Obtain hardware/firmware observations before marking any device feature verified.
