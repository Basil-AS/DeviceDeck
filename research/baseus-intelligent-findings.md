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

### Static transport and resource API findings

These findings describe shared app code, not commands confirmed on any specific target model.

- `BluetoothDataWriteManager` has a connection helper and three write paths. It selects classic Bluetooth for models matching `DeviceManager.m18316H0`; otherwise it uses the vendor `BleApi`. Inputs are model name, hex command text, and Bluetooth address. One overload adds a priority; another calls a distinct BLE SDK method. Whether those SDK methods map to GATT write, write-with-response, or different characteristics still requires SDK tracing and a capture.
- Before sending, `m18287b` checks a model predicate (`m18334Q0`). For selected models, commands not already prefixed `789C` are converted by `HeadPhoneDataResolveManager.m18972f`; other models pass the string through unchanged. The converter expects `BA` plus at least one command byte, builds a `789C`-prefixed envelope with lengths/selector bytes, and appends a CRC. Its selector table returns marker values such as `00`, `01`, or `03`; their complete semantics are not established. The response parser validates frame length and CRC and splits responses. It special-cases response codes `1B` and `4D`. Thus `BAxx` values seen at the app layer are not necessarily raw GATT payloads.
- The requested internal names (`Baseus Bass EP10 Pro`, `Bowie 30`, `Baseus Bowie MF1`, `Blade 2-140`) do not appear in either the decompiled classic-transport predicate (`m18316H0`) or the `789C` conversion predicate (`m18334Q0`). This means the shared methods above do not establish which path or framing any of these four targets use. In particular, do not apply the `789C` wrapper to them based only on another Baseus model.
- `ResourceManager` fetches per-model resources via HTTPS from `base-sz.oss-accelerate.aliyuncs.com`. `HomeFragmentPresenter` supplies a model name, `DeviceResVersion1` category/version, MD5, and resource URL. The app stores the downloaded data plus version and MD5 metadata. The product-resource package contents were checked live below; this endpoint currently serves presentation media and does not expose a command schema.
- `DeviceResVersion1`'s model-to-category mapping selects a resource family/version; it is not an opcode and does not prove that an operation is available or accepted by hardware.

### Dynamic product-resource API check (2026-11-08)

The app's `ControlApi.m15009S` calls `POST /app/devicebind/getProductResource` with `{"list":[{"model":...,"version":...}]}` against the configured regional Baseus API. A read-only request to the EU host returned entries for all four target models without a user account. The response and downloaded ZIPs are preserved locally (ignored by Git) in `research/local-evidence/baseus-resources/`; the API `md5` values matched the downloaded files.

| Model sent to API | Resource version | Returned name / category | Download ZIP contents | Meaning |
|---|---:|---|---|---|
| `Baseus Bass EP10 Pro` | 3 | `Bass EP10 Pro` / 15 | binding video, product PNG, find-lost GIF, map icon | Product presentation assets; no command table or control JSON in this package |
| `Bowie 30` | 2 | `Bowie 30` / 15 | binding video, product PNG, find-lost GIF, map icon | Same |
| `Baseus Bowie MF1` | 3 | `Baseus Bowie MF1` / 21 | binding video, product PNG, find-lost GIF, map icon | Same |
| `Blade 2-140` | 0 | `Blade 2-140` / 34 | add-device video and product PNG | Mobile-power presentation assets; no BLE control profile in this package |

The four API entries had `gestureUrl: null` and no per-model command/feature payload field. This endpoint therefore does **not** fill the missing protocol map; it serves product metadata and artwork/tutorial assets. Do not redistribute the downloaded vendor images/videos/GIFs. Keep Baseus support at research-only until command/state captures or another first-party protocol source identifies exact writes, notifications, and read-back for each model.

| Area | Static evidence | Still unknown |
|---|---|---|
| Transport | Model predicate selects BLE or classic; command input is hex; a model group uses a `789C` envelope and CRC. | Target-specific GATT UUIDs, characteristic mapping, and behavior of each vendor SDK write method. |
| Framing | Parser checks encoded lengths and CRC, splits responses, special-cases codes `1B` and `4D`. | Complete frame semantics, request/response mapping, and which requested models use each frame. |
| Model controls | Local model names, resource categories, and UI branches exist; resources may be fetched dynamically. | Exact target resources, feature-to-command mapping, state queries/decoding, and read-back/persistence. |
| UI choices | EP10 Pro has a special noise-resource type selector; Bowie 30 UI lists Off, Transparency, and Noise Reduction. | Whether every choice maps to an implemented write and is accepted by the matching model/firmware. |

Do not implement a generic `BAxx` value as a model feature until evidence links the UI action, exact model and resource version, encoded write, response/read-back, and hardware/firmware. The dynamic resource dependency must be included in repeatable analysis; do not treat an inferred URL or payload as a stable public protocol.

Before implementing commands, capture Baseus Intelligent while it reads state and changes one setting at a time. Preserve the service/characteristic UUIDs, request and response bytes, model/firmware pair, and the matching UI action. For Blade 2, separately establish whether its reported battery/current/charge statistics are available over BLE, and whether any advertised write is a charging-safety operation. Do not add writes for power delivery, battery protection, reset, or firmware update without a safe, verified protocol and explicit UI guard.

## Provenance pointers

- `DeviceManager.java`: exact model-name registry and grouped device predicates.
- `DeviceResVersion1.java`: model-to-resource category mapping.
- `NoiseReduceViewModel.java`: mode choices; `m49081z` includes `Bowie 30` in the transparency-capable group.
- `NoiseReduceManger.java`, `NoiseReduceManger2.java`: model-specific noise resource type selection, including EP10 Pro type 101.
- `gesture/gesture_config_function_list.json` and `_v2.json`: gesture/action function IDs and model eligibility.
- `MobilePowerUtil.java` and `view/activity/mobilepower/`: app-side Blade 2-140 monitoring screens.
- `BluetoothDataWriteManager.java`, `HeadPhoneDataResolveManager.java`, `HeadPhoneCrcUtil.java`: shared command conversion, framing, parsing, and CRC paths.
- `ResourceManager.java`, `DownloadManager.java`, `DownloadInterface.java`: per-model resource download/cache path.

All these pointers refer to the local decompilation of the version/hash above. Re-check them if the vendor app version changes.
