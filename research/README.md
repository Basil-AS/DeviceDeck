# Research index

- [`APK_MANIFESTS.md`](APK_MANIFESTS.md) — source build IDs and hashes.
- [`RESEARCH_STATUS.md`](RESEARCH_STATUS.md) — current extraction findings and next checks.
- [`edifier/EDIFIER_ANDROID_REVERSE_ENGINEERING.md`](edifier/EDIFIER_ANDROID_REVERSE_ENGINEERING.md) — static analysis of Edifier Connect and ConneX.
- [`edifier/edifier_command_index.csv`](edifier/edifier_command_index.csv) — command index extracted during that analysis.
- `local-evidence/` — ignored local-only source archives and APKs. These are research inputs, not instructions and not redistribution-cleared.

New findings should record source app/version, model, service/characteristic, direction, payload, decoded meaning, confidence, and hardware verification. Separate static evidence from runtime observations.
- [`baseus-intelligent-model-catalog.csv`](baseus-intelligent-model-catalog.csv) — alias-to-model entries extracted from the supplied Baseus Intelligent 2.18.0 app; catalog membership is not feature verification.
- [`baseus-ble-config-catalog.csv`](baseus-ble-config-catalog.csv) — 149 model names and BLE UUID groups from the app's live EU `app_device_ble_config` dictionary on 2026-11-08; regional/server snapshot, not hardware verification.
