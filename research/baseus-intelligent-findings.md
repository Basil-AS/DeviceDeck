# Baseus Intelligent: requested-device findings

**Source:** user-supplied Baseus Intelligent 2.18.0 (version code 185), arm64 split bundle. Bundle SHA-256: `c93512f905fca24d401d0c45d8e795a1e37db96864d0489fc051d437a53f4003`. Decompiled output is local-only under `research/local-evidence/decompiled/baseus/`; no APK, vendor art, or full proprietary resource is redistributed here.

## What the static app evidence establishes

**Correction to the earlier protocol boundary:** the earlier pass stopped at the product-resource API and shared `789C` framing predicate. That was incomplete. The APK contains model-specific noise-mode UI and writes `BA34` through the shared writer. The four models not being in `m18334Q0` means they are *not* wrapped by that `789C` path; it does not mean they lack command support. Transport is model-dependent: EP10 Pro and Bowie MF1 are selected by classic RFCOMM predicate `m18316H0`; Bowie 30 and Blade 2 use the BLE branch in `BluetoothDataWriteManager` (Blade 2 is a separate power-device flow). Do not assume all target `BAxx` commands go through GATT.

The app recognizes these exact internal model names and has app-side product/resource mappings for them:

| Requested product | Internal model string | Static evidence | App-side control evidence | DeviceDeck status |
|---|---|---|---|---|
| Baseus EP10 Pro | `Baseus Bass EP10 Pro` | `DeviceManager` model map; resource category 3 | Noise UI queries `BA33` and writes mode via `BA34 <mode> <level>`; app selects noise type 101 for this model. Gesture UI/config is present, but gesture wire mapping is not resolved. | Research only |
| Bowie 30 | `Bowie 30` | Resource category 2 | Noise UI offers Off (0), Transparency (2), Noise Reduction (1); common writer queries `BA33`/`BA35` and writes `BA34 <mode> <level>`. Gesture UI/config is present, but gesture wire mapping is not resolved. | Research only |
| Bowie MF1 | `Baseus Bowie MF1` | `DeviceManager` model map; resource category 3 | Gesture UI/config is present. No explicit noise mode entry was found for this model in inspected selectors; do not infer ANC. | Research only |
| Baseus Blade 2 | `Blade 2-140` | Resource category 0; `DeviceManager` model row | Dedicated mobile-power UI/view-model supports version and charge/statistics views. Exact telemetry requests and any safe writes are not yet resolved. | Research only |

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
- Before sending, `m18287b` checks a model predicate (`m18334Q0`). For selected models, commands not already prefixed `789C` are converted by `HeadPhoneDataResolveManager.m18972f`; other models pass the string through unchanged. The converter expects `BA` plus at least one command byte, builds a `789C`-prefixed envelope with lengths/selector bytes, and appends a CRC. Its selector table returns marker values such as `00`, `01`, or `03`; their complete semantics are not established. The response parser validates frame length and CRC and splits responses. It special-cases response codes `1B` and `4D`.
- **Classic transport:** `DeviceManager.m18316H0` returns true for the exact names `Baseus Bass EP10 Pro` and `Baseus Bowie MF1`. `BluetoothDataWriteManager` therefore dispatches their writes to `ClassBt`, not `BleApi`. `ClassicBluetoothManager` constructs its RFCOMM connection with UUID `00001101-0000-1000-8000-00805F9B34FB` (SPP). **BLE transport:** `Bowie 30` and `Blade 2-140` are absent from this predicate and take the `BleApi` branch; the exact GATT endpoints still depend on the separate BLE configuration described below.
- The requested names are absent from the `789C` conversion predicate (`m18334Q0`), so the command strings for all four targets are not wrapped by that shared encoder. Static noise builders use `BA34` followed by two bytes: mode and level/type (`FF` means unspecified); EP10 Pro normalizes vendor mode `11` to wire mode `01`. The noise UI sends `BA33` and `BA35` when requesting current noise state/config. These are app-side command facts, not independent hardware-confirmed behavior.
- Blade 2 is a mobile-power profile, not an earbud profile. `DeviceManager.m18315H` excludes Blade 2 from the ordinary headphone path. Keep its power telemetry and charging controls on a separate protocol/session and expose no write until its operation and safety semantics are established.
- BLE service/write/notify UUIDs are not hard-coded in the APK for every model. `HomeFragmentPresenter.m48034J1` maps the server-provided `DeviceBleConfigBean` fields (`service_uuid`, `write_characterristic_uuid`, `notify_characterristic_uuid`) into the BLE manager. A separate model list enables the SDK's “use first GATT service, first two characteristics” fallback. This is relevant to Bowie 30 and Blade 2, not to the SPP path used by EP10 Pro and MF1. The product-resource endpoint examined below is a different API and does not return this BLE transport configuration.
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

The four API entries had `gestureUrl: null` and no per-model command/feature payload field. This endpoint therefore does **not** supply the BLE transport configuration; it serves product metadata and artwork/tutorial assets. The app's separate `DeviceBleConfigBean` flow supplies service/characteristic UUIDs. Do not redistribute the downloaded vendor images/videos/GIFs. DeviceDeck remains research-only until it has a transport configuration and a read/write/readback trace for each model.

| Area | Static evidence | Still unknown |
|---|---|---|
| Transport | EP10 Pro/MF1: Android SPP RFCOMM using the standard SPP UUID. Bowie 30/Blade 2: Baseus BLE SDK. `789C` selector is false for all four. | Windows/macOS RFCOMM interoperability and Bowie 30/Blade 2 GATT UUID/config behavior need platform-specific verification. |
| Framing | Earbud noise commands use `BAxx` app-layer bytes; `BA34 <mode> <level>` is generated by the UI helper and written unchanged for these target names. | Hardware confirmation, notification/state parser mapping, and write completion/readback per model/firmware. |
| Model controls | EP10 Pro and Bowie 30 have specific noise-mode UI/query/write paths; requested gesture and Blade UI paths are also present. | Gesture function IDs to wire bytes; MF1 mode support; Blade telemetry request/response and any safe controls. |
| UI choices | EP10 Pro has a special noise-resource type selector; Bowie 30 UI lists Off, Transparency, and Noise Reduction. | Whether every choice maps to an implemented write and is accepted by the matching model/firmware. |

Do not implement a generic `BAxx` value as a model feature until evidence links the UI action, exact model and resource version, encoded write, response/read-back, and hardware/firmware. The dynamic resource dependency must be included in repeatable analysis; do not treat an inferred URL or payload as a stable public protocol.

Before claiming working support, capture Baseus Intelligent while it reads state and changes one setting at a time. Preserve the service/characteristic UUIDs, request and response bytes, model/firmware pair, and the matching UI action. For Blade 2, separately establish whether its reported battery/current/charge statistics are available over BLE, and whether any advertised write is a charging-safety operation. Do not add writes for power delivery, battery protection, reset, or firmware update without a safe, verified protocol and explicit UI guard.

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
