# Baseus Intelligent: requested-device findings

**Source:** user-supplied Baseus Intelligent 2.18.0 (version code 185), arm64 split bundle. Bundle SHA-256: `c93512f905fca24d401d0c45d8e795a1e37db96864d0489fc051d437a53f4003`. Decompiled output is local-only under `research/local-evidence/decompiled/baseus/`; no APK, vendor art, or full proprietary resource is redistributed here.

## What the static app evidence establishes

**Correction to the earlier protocol boundary:** the earlier pass stopped at the product-resource API and shared `789C` framing predicate. That was incomplete. The APK contains model-specific noise-mode UI and writes `BA34` through the shared writer. The four models not being in `m18334Q0` means they are *not* wrapped by that `789C` path; it does not mean they lack command support. Transport is model-dependent: EP10 Pro and Bowie MF1 are selected by classic RFCOMM predicate `m18316H0`; Bowie 30 and Blade 2 use the BLE branch in `BluetoothDataWriteManager` (Blade 2 is a separate power-device flow). Do not assume all target `BAxx` commands go through GATT.

The app recognizes these exact internal model names and has app-side product/resource mappings for them:

| Requested product | Internal model string | Static evidence | App-side control evidence | DeviceDeck status |
|---|---|---|---|---|
| Baseus EP10 Pro | `Baseus Bass EP10 Pro` | `DeviceManager` model map; resource category 3 | Noise UI queries `BA33` and writes mode via `BA34 <mode> <level>`; app selects noise type 101 for this model. The v2 gesture template and BA8C–BA8F query/set command shapes are statically resolved. | Implemented in the unified Android, Windows/Linux, and macOS hubs: SPP battery/noise controls, gesture assignments and cycle-mode children with readback; hardware round-trip remains pending. |
| Bowie 30 | `Bowie 30` | Resource category 2 | Noise UI offers Off (0), Transparency (2), Noise Reduction (1); common writer queries `BA33`/`BA35` and writes `BA34 <mode> <level>`. Gesture UI/config exists, but gesture wire mapping is not resolved. | Implemented in the unified apps for BLE battery/noise controls; mode coverage varies by platform and needs hardware checks. Gesture writes remain unavailable. |
| Bowie MF1 | `Baseus Bowie MF1` | `DeviceManager` model map; resource category 3 | Legacy gesture UI/config lists this exact model. It uses BA21/AA21 layout query and BA22/AA22 paired left/right assignment write; no explicit noise mode entry was found, so do not infer ANC. | Battery and model-gated legacy gesture query/action picker with acknowledgement and readback are integrated in the unified Android, Windows/Linux, and macOS apps. Physical-device validation remains open. |
| Baseus Blade 2 | `Blade 2-140` | Resource category 0; `DeviceManager` model row | Dedicated mobile-power UI/view-model supports version and charge/statistics views. Static telemetry and display/timer/port/reset commands are mapped in the sections below. | Implemented in the unified apps with telemetry, display/timer controls, C1/C2 mode controls, Blade 2-140 cell metrics, and an explicit-confirmation reset action. Reset has no observed acknowledgement/readback; device acceptance, scaling, reset outcome, and port behavior still need hardware checks. |

`DeviceResVersion1` returns resource categories, not protocol command IDs. Its category-0 fallback also recognizes entries from the app's energy-storage enum. The product resource mapping alone must not be treated as proof that controls are bundled or work offline.

## Blade 2-140: extracted mobile-power functionality

The decompiled app contains substantially more than a battery percentage screen for this model. `MainFragmentViewModel` and `StatisticsFragmentViewModel` build the requests and parse the responses. The following is **static app evidence** from Baseus Intelligent 2.18.0 (185); a live capture against the user's Blade 2 is still needed to prove firmware acceptance, scale factors and exact meaning of each port-status bit.

### UI features found

- Main page: temperature, electricity/energy value, remaining time, charge time, port status/error, reminder and auto-off countdowns, screen theme, screen-saver state and user text; Bluetooth/BMS/DC module versions are also queried/displayed.
- Screen personalization: three theme presets (the UI calls them themes; do not label them as an arbitrary RGB color picker), screen-saver toggle and custom UTF-8 text. Text input is filtered by the app and sent as UTF-8 hex. The code trims whitespace and ignores empty strings. The exact device-side character/byte limit is not safely inferable from the UI code alone.
- Statistics page: Type-C 1 and Type-C 2 each display voltage, current, power and negotiated charging protocol; the app maintains separate input/charge and output/discharge readings. It also renders six battery-cell voltage/current/status entries, cell temperature and cycle count, plus a repair indicator. This is the source for the requested per-port voltage readings.
- Controls: per-port enable/disable actions are built for two Type-C ports. The UI also supports reminder and shutdown countdowns. Reset-to-default exists in the main view-model; do not expose it without a deliberate confirmation.
- Polling: statistics activates two periodic control writes/requests while the statistics view is open and disables them on exit. The app polls the port state during an active session. Treat codes 0037/0038 as opaque polling/enable controls until captured; their semantic names are not established here.

### App-layer request format and code table

The BLE view-model constructs reads as `9AAA03 + code + 0001` and writes as `9AAA10 + code + 0001 + 0201 + value`, then passes the full string through `EnergyStorageUtil`'s CRC16 routine. These are application payloads; the BLE library still owns characteristic transport/framing.

| Code | App-side interpretation/action | Evidence level |
|---|---|---|
| `0001`–`000C` | Type-C1 output V/A/W (`0001`/`0002`/`0003`); Type-C2 output V/A/W (`0004`/`0005`/`0006`); Type-C1 input V/A/W (`0007`/`0008`/`0009`); Type-C2 input V/A/W (`000A`/`000B`/`000C`). Parser computes raw value divided by the response multiplier byte. | Exact app parser mapping; scale/range needs device capture |
| `000D` | Main-page temperature, raw minus 100 | Exact parser mapping |
| `0010` | Port/status bitfield. The low two bits are C1 state and bits 2–3 are C2 state: `0` off, `1` output/discharge, `2` input/charge. Higher bits also carry cell status. | Exact static parser/use and mode values |
| `0011` | Device error value | Exact parser mapping; error enum not yet decoded |
| `0012` | Type-C charging protocol code | Exact parser mapping; label table in `MobilePowerUtil` |
| `0013`, `0023`, `0024` | Bluetooth module, BMS and DC module versions | Exact app read/display behavior |
| `0014` | Main-page electricity/energy value | Exact parser mapping; physical unit/scaling needs capture/UI string cross-check |
| `0015`, `0016` | Remaining time and charge time; app caps values at 1440 | Exact parser mapping |
| `0018`–`001B` | Port writes: C1 output/discharge=`0018`, C1 input/charge=`0019`, C2 output/discharge=`001A`, C2 input/charge=`001B`. `0000` clears the selected mode; `0001` enables it. The app clears the prior mode before enabling a replacement, then rereads the status. | Exact mapping from the decompiled `StatisticsFragmentViewModel` parser and its `W0`/`X0` transition methods; the physical connector labels and firmware behavior still need a live-device check |
| `001E`, `001F` | Reminder countdown and automatic shutdown countdown | Exact read/write path |
| `0020` | Screen theme; UI selects one of three indices (0, 1, 2) and writes a four-hex-digit value | Exact UI choices/write path |
| `0021` | A write path exists in the main view-model, but its UI meaning was not established in this pass | Known write builder, semantic unresolved |
| `0022` | Screen-saver state | Exact parser/write path |
| `0025`, `0026` | Blade 2-140 cell temperature (raw minus 100) and cycle count | Exact app parser/read list |
| `0028`–`002D` | Battery-cell voltage values 1–6 | Exact statistics parser mapping |
| `0030` | Custom screen text, decoded/encoded as UTF-8 | Exact parser and write path |
| `0031`–`0036` | Battery-cell current values 1–6 | Exact statistics parser mapping |
| `0037`, `0038` | Enabled while statistics screen is active; turned off when leaving | Exact lifecycle call; command semantics unresolved |
| `000E` | Restore defaults write with value `0000` | Exact reset path; potentially disruptive, keep gated |

`MobilePowerUtil` maps protocol integers to labels including PPS, PD 3.0, QC 2.0/3.0, AFC, FCP and SCP; Blade2-140 uses a model-specific index table. The table indicates what the app can display and does not guarantee every protocol is supported by every port/accessory.

### Important correction to earlier project notes

The previous sentence that Blade telemetry requests and safe writes were “not yet resolved” is now superseded for *static app logic*: the view-model constructs and parses the controls above. Hardware-level confirmation remains outstanding. The exact BLE UUID/config is not bundled in the product-resource ZIP; it is delivered by the app's separate BLE configuration path. Consequently, a standalone implementation still needs a captured device configuration (or safe discovery procedure), notification/readback trace and firmware/model identifier.

## Baseus app device families beyond headphones

The APK contains dedicated app modules for these families (folder/module presence is not proof every model is currently sold, supported in every region or controllable offline):

| Family/module | What is visible from app structure | Confidence/next evidence |
|---|---|---|
| Earphones/headphones | TWS and over-ear controls: battery, ANC/transparency, EQ/sound effects, touch gestures, find-device, firmware/account/content adjuncts; model feature availability differs | Many static model mappings; individual protocol profiles need work |
| Mobile power / power banks | Blade 2-140 dashboard, screen themes/text, timeouts, status and detailed port/cell telemetry described above | Strong UI/protocol-builder evidence; live BLE trace required |
| Portable power stations | Separate NRG and PES600W controllers, home/output/wave/settings/parallel-operation screens, BLE broadcast receiver and update manager | Explicit controllers for PES600W, BPM600W, PPS140W, PPS140, PPS450W, PPS420W; per-model command audit remains |
| Smart sockets/receptacles | Dedicated Wi-Fi device flow | Cloud/account and exact model capabilities need inspection |
| Charging products | Charging station, charging gun, “charging nebula” modules | App modules exist; do not equate this to BLE support without model registry/protocol trace |
| Home and pet appliances | Purifier, pet purifier/water, fridge and washing-machine modules | Presence in app; likely mixed cloud/local protocols, model list not fully resolved |
| Scent and environment products | Scent machine, double scent, cup scent; sleep-alarm modules | Presence in app; device/function mapping still needed |
| Other accessories/services | Smart mouse, stylus, gesture devices, Wi-Fi provisioning, “third serve”, XMly integration | App modules/services; not all are standalone BLE accessories |

The complete module directory inventory and confidence per family are recorded in [`baseus-device-families.csv`](baseus-device-families.csv). The existing [`baseus-intelligent-model-catalog.csv`](baseus-intelligent-model-catalog.csv) is specifically a partial static *earbud name registry* plus the four requested product-resource mappings, not a complete all-Baseus product catalog. Server-driven add-device catalogs are region/account/runtime dependent, so the app package alone cannot provide a defensible complete supported-model list.

## Gesture function IDs

The bundled `gesture_config_function_list.json` maps these numeric function IDs to eligible actions. The IDs below come from that older list; the EP10 Pro v2 resource and command format are documented separately below. None of these statically recovered settings has been validated on physical hardware.

| Model | Single click | Double click | Long press | Triple click |
|---|---|---|---|---|
| EP10 Pro | `[1, 0]` | `[1, 6, 2, 3, 4, 11, 12, 0]` | `[1, 6, 2, 3, 4, 11, 12, 0]` | `[1, 6, 2, 3, 4, 11, 12, 0]` |
| Bowie 30 | No model-specific entry in this file | `[1, 3, 2, 4, 6, 0]` | `[1, 3, 2, 4, 6, 0]` | `[4, 5]` |
| Bowie MF1 | No model-specific entry in this file | `[11, 12, 1, 2, 3, 4, 0]` | `[11, 12, 1, 2, 3, 4, 0]` | `[4, 5]` |
| Blade 2 | Not an earbud gesture profile | Not an earbud gesture profile | Not an earbud gesture profile | Not an earbud gesture profile |

The v2 gesture resource also contains EP10 Pro entries with a `functionId: 6` child-value list `[1, 2, 0]`. The app resolves function 6 as a noise-mode action; child IDs map to Normal (0), ANC (1), and Transparency (2). This is app UI semantics; device acceptance still needs a hardware capture.

### Bowie MF1 legacy gesture command format (static app evidence)

Unlike EP10 Pro's v2 BA8C–BA8F mapping, the bundled legacy `gesture_config_layout.json` and `gesture_config_function_list.json` list `Baseus Bowie MF1` with layouts 0 (double click), 1 (triple click), and 2 (long press). The legacy app path in `GestureBleManager` builds `BA21 <layout>` to query and `BA22 <layout> <left action> <right action>` to set a layout. `GestureSettingViewModel` parses `AA21 <layout> <left action> <right action>` and `AA22 <result>`; `GestureSettingActivity` treats result `01` as success. The SPP transport predicate includes the exact MF1 name, so the shared writer uses RFCOMM for these commands.

The model's app-listed function IDs are layout-specific: double/long `[11,12,1,2,3,4,0]`; triple `[4,5]`. `GestureDataResolveManager` resolves the legacy IDs as quick call, pairing mode, battery announcement, play/pause, answer/end call, reject call, power on/off, and quick listen. DeviceDeck only offers those model-listed values, preserves the other side's current assignment, requires the success response, then rereads the layout and checks the changed side. This mapping is derived from static app resources/code; no live MF1 packet capture or hardware round-trip is recorded. The feature is present in all unified platform apps but remains marked unverified until tested with a named MF1 firmware.

### EP10 Pro v2 gesture command format (static app evidence)

`GestureSettingViewModel` builds these ASCII-hex command strings and passes them to `BluetoothDataWriteManager`. EP10 Pro is selected for classic RFCOMM/SPP (`DeviceManager.H0`), and it is not in the `789C` wrapper predicate (`DeviceManager.Q0`), so the shared writer passes the command bytes through unchanged before sending. The response prefixes below are parsed by the app. This is a statically recovered app protocol, not a live-device trace.

| Operation | Request bytes (hex) | App response handling |
|---|---|---|
| Query a button's gesture bindings | `BA 8C <button> <layout>`; use `FF` for all layouts | `AA 8C <button> <layout/function pairs...>`; each remaining two-byte pair is layout ID then function ID |
| Set one gesture binding | `BA 8D <button> <layout> <function>` | `AA 8D` is handled as a setting result |
| Query options for a function with child choices | `BA 8E <button> <layout> <function>` | `AA 8E <button> <layout> <function> <count> <child IDs...>` |
| Set child choices | `BA 8F <button> <layout> <function> <count> <child IDs...>`; zero children is encoded with count `00` | `AA 8F` is handled as a setting result |

The model template maps button `00` to left and `01` to right. Layout IDs are `03` single click, `00` double click, `01` triple click, and `02` long press (from the bundled layout config and its layout-label resolver). Default function IDs are left `{single:0,double:1,long:6,triple:2}` and right `{single:0,double:1,long:6,triple:3}`. The bundled action-option list gives single-click `[1,0]`; double, long, and triple click `[1,6,2,3,4,11,12,0]`. Function `6` has child IDs `[1,2,0]`, displayed as ANC, Transparency, and Normal. These profile values are UI choices, not a claim that every binding works on every EP10 Pro firmware.

The status response `AA 8B <supported>` is decoded as the v2-gesture capability flag (`01` means supported). The exact lifecycle/trigger for that capability query still needs a second pass through activity initialization. DeviceDeck's Android, Windows/Linux, and macOS hubs use BA8C/BA8D to read assignments, offer primary-action choices, and verify writes by reading them back. BA8E/BA8F child-choice editing for the “Cycle noise modes” action is also implemented across these hubs; all mappings still need live read/write/readback capture on the target headphones.

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

The four API entries had `gestureUrl: null` and no per-model command/feature payload field. This endpoint therefore does **not** supply the BLE transport configuration; it serves product metadata and artwork/tutorial assets. The separate `DeviceBleConfigBean` flow is documented below. Do not redistribute the downloaded vendor images/videos/GIFs. A UUID profile is not a read/write/readback trace, so each target remains hardware-unverified.

### Live BLE configuration dictionary (2026-11-08)

The other configuration path is now identified: Baseus calls `GET /app/homepage/dictByName?dictName=app_device_ble_config`. A read-only request to the public EU API host `https://bds-api-eu.baseus.com` returned a successful dictionary without an account. The APK reads each dictionary value as `DeviceBleConfigBean` and passes model names and UUID triples to its BLE SDK. The generated snapshot is [`baseus-ble-config-catalog.csv`](baseus-ble-config-catalog.csv); it contains **149 exact model-name entries across five config groups** at retrieval time.

| Group (`dictSort`) | Entries | Service UUID | Write UUID | Notify UUID | Examples |
|---:|---:|---|---|---|---|
| 1 | 26 | `EDFEC62E-9910-0BAC-5241-D8BDA6932A2F` | `2D86686A-53DC-25B3-0C4A-F0E10C8AEC21` | `15005991-B131-3396-014C-664C9867B917` | T1/T2/T3, Bowie E8, Encok W04/W11/W12 |
| 2 | 2 | `0000FFF0-0000-1000-8000-00805F9B34FB` | `0000FFF2-0000-1000-8000-00805F9B34FB` | `0000FFF1-0000-1000-8000-00805F9B34FB` | Baseus X3, X5 |
| 3 | 6 | `0000AE30-0000-1000-8000-00805F9B34FB` | `0000AE01-0000-1000-8000-00805F9B34FB` | `0000AE02-0000-1000-8000-00805F9B34FB` | PES600W, PPS450W, PPS140, BPM600W, BS-B08, BS-GaN240 |
| 4 | 114 | `53527AA4-29F7-AE11-4E74-997334782568` | `EE684B1A-1E9B-ED3E-EE55-F894667E92AC` | `654B749C-E37F-AE1F-EBAB-40CA133E3690` | Blade 2/Blade 2-140, Bowie 30/MF1, EP10 Pro, BP1 Pro/Ultra and many others |
| 5 | 1 | `00001234-0000-1000-8000-00805F9B34FB` | `00001235-0000-1000-8000-00805F9B34FB` | `00001236-0000-1000-8000-00805F9B34FB` | BS-TS015 |

The requested products resolve in group 4 to the same service/write/notify UUID triple as the BP1 Pro/Ultra profiles already used by the Baseus desktop fork. This is strong transport-profile evidence for Blade 2-140, Bowie 30, Bowie MF1 and EP10 Pro. It does **not** prove that model-specific app commands, advertisement matching or readback work; in particular, Blade 2 must still be distinguished from earbuds sharing those UUIDs. Treat this API as a live regional catalogue, not a frozen protocol guarantee. The CSV is a dated observed snapshot, with no API request identifiers or account data saved.

| Area | Static evidence | Still unknown |
|---|---|---|
| Transport | EP10 Pro/MF1: Android SPP RFCOMM using the standard SPP UUID. Bowie 30/Blade 2: Baseus BLE SDK. `789C` selector is false for all four. Live EU BLE dictionary now gives a common service/write/notify UUID triple for all four targets. | Windows/macOS SPP interoperability and model advertisement matching, write/readback over the BLE profile need platform-specific verification. |
| Framing | Earbud noise commands use `BAxx` app-layer bytes; `BA34 <mode> <level>` is generated by the UI helper and written unchanged for these target names. | Hardware confirmation, notification/state parser mapping, and write completion/readback per model/firmware. |
| Model controls | EP10 Pro and Bowie 30 have specific noise-mode UI/query/write paths. EP10 Pro BA8E reads the selected children for action 6 (“cycle noise modes”); BA8F writes the selected IDs. DeviceDeck exposes these choices in the unified controls on Android, Windows/Linux, and macOS with a minimum of two selections and BA8E readback. Blade app-layer read/write map and live GATT profile are listed above. | Physical-device validation for gesture writes; MF1 mode support; Blade response/readback and firmware validation. |
| UI choices | EP10 Pro has a special noise-resource type selector; Bowie 30 UI lists Off, Transparency, and Noise Reduction. | Whether every choice maps to an implemented write and is accepted by the matching model/firmware. |

Do not implement a generic `BAxx` value as a model feature until evidence links the UI action, exact model and resource version, encoded write, response/read-back, and hardware/firmware. The dynamic resource dependency must be included in repeatable analysis; do not treat an inferred URL or payload as a stable public protocol.

Before claiming working support, capture Baseus Intelligent while it reads state and changes one setting at a time. Preserve the service/characteristic UUIDs, request and response bytes, model/firmware pair, and the matching UI action. For Blade 2, separately establish whether its reported battery/current/charge statistics are available over BLE, and whether any advertised write is a charging-safety operation. Do not add writes for power delivery, battery protection, reset, or firmware update without a safe, verified protocol and explicit UI guard.

## Provenance pointers

- `DeviceManager.java`: exact model-name registry and grouped device predicates.
- `DeviceResVersion1.java`: model-to-resource category mapping.
- `NoiseReduceViewModel.java`: mode choices; `m49081z` includes `Bowie 30` in the transparency-capable group.
- `NoiseReduceManger.java`, `NoiseReduceManger2.java`: model-specific noise resource type selection, including EP10 Pro type 101.
- `gesture/gesture_config_function_list.json` and `_v2.json`: gesture/action function IDs and model eligibility.
- `MobilePowerUtil.java`, `view/activity/mobilepower/viewmodel/MainFragmentViewModel.java`, `StatisticsFragmentViewModel.java`, and `MobilePowerThemeChooseActivity.java`: Blade 2-140 screens, app-layer command builders/parser, telemetry and screen customization.
- `view/activity/energystorage/strategy/` and `utils/util_ble/BleCommandConst.java`: power-station controller registry and family-specific command handlers.
- `BluetoothDataWriteManager.java`, `HeadPhoneDataResolveManager.java`, `HeadPhoneCrcUtil.java`: shared command conversion, framing, parsing, and CRC paths.
- `ResourceManager.java`, `DownloadManager.java`, `DownloadInterface.java`: per-model resource download/cache path.
- `ControlApi.java` `m15025a`, `HomeFragmentPresenter.mo46335G0`, and `m48034J1`: live `app_device_ble_config` dictionary fetch and registration of the model-name-to-GATT UUID map.

All these pointers refer to the local decompilation of the version/hash above. Re-check them if the vendor app version changes.
