# Baseus Intelligent: requested-device findings

**Source:** user-supplied Baseus Intelligent 2.18.0 (version code 185), arm64 split bundle. Bundle SHA-256: `c93512f905fca24d401d0c45d8e795a1e37db96864d0489fc051d437a53f4003`. Decompiled output is local-only under `research/local-evidence/decompiled/baseus/`; no APK, vendor art, or full proprietary resource is redistributed here.

## What the static app evidence establishes

The app recognizes these exact internal model names and has app-side product/resource mappings for them:

| Requested product | Internal model string | Static evidence | App-side control evidence | DeviceDeck status |
|---|---|---|---|---|
| Baseus EP10 Pro | `Baseus Bass EP10 Pro` | `DeviceManager` model map; `DeviceResVersion1.m18018b` returns resource category 3 | Gesture config lists function IDs by gesture (below). `NoiseReduceManger` and `NoiseReduceManger2` special-case the model with type 101; this is a model/version selector, not a BLE opcode. | Research only |
| Bowie 30 | `Bowie 30` | `DeviceResVersion1.m18018b` returns category 2 | Noise UI builds Off (0), Transparency (2), and Noise Reduction (1) choices for the model. Gesture config has double, long, and triple click maps. | Research only |
| Bowie MF1 | `Baseus Bowie MF1` | `DeviceManager` model map; `DeviceResVersion1.m18018b` returns category 3 | Gesture config has double, long, and triple click maps. No explicit noise mode entry for this name was found in the inspected mode switches. That absence does not prove the device lacks ANC. | Research only |
| Baseus Blade 2 | `Blade 2-140` | `DeviceResVersion1.m18018b` returns category 0; `DeviceManager` has a model row | Dedicated mobile-power UI/view-model code checks `Blade 2-140` for version and charging/statistics views. This shows app-side product recognition, not yet an independent BLE-control profile. | Research only |

`DeviceResVersion1` returns resource categories, not protocol command IDs. Its category-0 fallback also recognizes entries from the app's energy-storage enum. The product resource mapping alone must not be treated as proof that controls are bundled or work offline.

## Gesture function IDs

The bundled `gesture_config_function_list.json` maps these numeric function IDs to eligible actions. The IDs below are copied as IDs; semantic labels and wire encodings still need a verified resource-to-command trace.

| Model | Single click | Double click | Long press | Triple click |
|---|---|---|---|---|
| EP10 Pro | `[1, 0]` | `[1, 6, 2, 3, 4, 11, 12, 0]` | `[1, 6, 2, 3, 4, 11, 12, 0]` | `[1, 6, 2, 3, 4, 11, 12, 0]` |
| Bowie 30 | No model-specific entry in this file | `[1, 3, 2, 4, 6, 0]` | `[1, 3, 2, 4, 6, 0]` | `[4, 5]` |
| Bowie MF1 | No model-specific entry in this file | `[11, 12, 1, 2, 3, 4, 0]` | `[11, 12, 1, 2, 3, 4, 0]` | `[4, 5]` |
| Blade 2 | Not an earbud gesture profile | Not an earbud gesture profile | Not an earbud gesture profile | Not an earbud gesture profile |

The v2 gesture resource also contains EP10 Pro entries with a `functionId: 6` child-value list `[1, 2, 0]`. Resolve that ID against app strings and the runtime profile before exposing controls.

## Protocol boundary and next capture

The app has model recognition in `com/base/module_common/manager/DeviceManager.java`, product resource versioning in `com/base/module_common/constant/control/DeviceResVersion1.java`, and gesture/noise UI logic under `com/control_center/intelligent/view/`. Home and add-device flows request/call `ResourceManager` and `ControlImpl`; the decompiled app has not yet established a complete offline packet table for these target devices.

Before implementing commands, capture Baseus Intelligent while it reads state and changes one setting at a time. Preserve the service/characteristic UUIDs, request and response bytes, model/firmware pair, and the matching UI action. For Blade 2, separately establish whether its reported battery/current/charge statistics are available over BLE, and whether any advertised write is a charging-safety operation. Do not add writes for power delivery, battery protection, reset, or firmware update without a safe, verified protocol and explicit UI guard.

## Provenance pointers

- `DeviceManager.java`: exact model-name registry and grouped device predicates.
- `DeviceResVersion1.java`: model-to-resource category mapping.
- `NoiseReduceViewModel.java`: mode choices; `m49081z` includes `Bowie 30` in the transparency-capable group.
- `NoiseReduceManger.java`, `NoiseReduceManger2.java`: model-specific noise resource type selection, including EP10 Pro type 101.
- `gesture/gesture_config_function_list.json` and `_v2.json`: gesture/action function IDs and model eligibility.
- `MobilePowerUtil.java` and `view/activity/mobilepower/`: app-side Blade 2-140 monitoring screens.

All these pointers refer to the local decompilation of the version/hash above. Re-check them if the vendor app version changes.
