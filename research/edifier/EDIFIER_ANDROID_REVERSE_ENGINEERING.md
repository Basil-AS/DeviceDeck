# Static reverse engineering: Edifier Connect and EDIFIER ConneX

Date of analysis: 2026-11-07. This report describes the supplied APKs, not a live Bluetooth capture.

## Scope and confidence

- **Edifier Connect**: package `com.edifier.edifierconnect`, version `8.4.28`, versionCode `272`; source reconstructed from the supplied split RAR archive. JADX reported 190 decode errors, so some decompiled fragments may be incomplete.
- **EDIFIER ConneX**: package `com.edifier.edifierconnex`, version `1.0.32`, versionCode `73`; source reconstructed from the supplied arm64 APKS set.
- Both include a vendor command library, BLE transport, product capability parsing, and UI code. There is no public SDK/API: the device-facing API is Edifier's private command protocol over GATT (or SPP for products whose database profile selects it).
- No Android phone/headset was attached during analysis. Static code proves that a command is implemented or called conditionally; it does not prove a particular W820NB Plus firmware accepts it.
- The complete command-index inventories are in [edifier_command_index.csv](./edifier_command_index.csv): Connect has 187 named indices, ConneX 179, union 197. This is the **generic product library**, not a claim that W820 supports every command.

## High-level connection path

1. The app loads a product catalog and matches a BLE advertisement to `ProductSearchUuid`. W820 family search UUIDs in Connect include:

   | Product | Search UUID |
   | --- | --- |
   | W820NB | `00003800-0000-1000-8000-00805f9b34fb` |
   | W820NB Classics | `0000a100-0000-1000-8000-00805f9b34fb` |
   | W820NB Gold | `00007900-0000-1000-8000-00805f9b34fb` |
   | W820NB Plus | `0000a500-0000-1000-8000-00805f9b34fb` |
   | W820NB T/G | `00003501-0000-1000-8000-00805f9b34fb` |

   These are discovery/model IDs, **not** the GATT service or characteristic UUIDs.
2. Connect checks the classic A2DP relationship and uses manufacturer data (manufacturer ID `0x07E0` when present) to associate the BLE control endpoint with the audio headset and configure protocol version/encryption. It scans at low latency, retries, and can require Android location/Bluetooth permissions depending on OS version.
3. The model catalog supplies `ProductServiceUuid`, `ProductReadUuid`, and `ProductWriteUuid`. The app opens GATT, requests MTU 500, discovers the configured service, enables notifications on the read characteristic, and writes frames to the write characteristic. The UUID catalog is carried in product data/database and is not fully readable as plain text from the supplied APK resources; do not substitute the search UUID for these values.
4. BLE notification bytes are accumulated/reassembled, CRC checked, decoded into command objects, persisted to the local `CommandTable`, then routed to UI state parsers.
5. Some product profiles select SPP instead of GATT. The database decides transport; the command API is shared.

ConneX's BLE implementation follows the same shape: it reads the three GATT UUIDs from its `ProductTable`, subscribes to notifications, chooses write-with/without-response from characteristic properties, and pushes data through its command layer. Relevant sources: [Connect BLEManager](file:///tmp/edifier-rev/connect/decompiled/sources/com/edifier/product_connect/connect/ble/BLEManager.java), [ConneX BLEManager](file:///tmp/edifier-rev/connex/sources/com/edifier/lib_connect/connect/ble/BLEManager.java).

## Wire format

Both libraries use header byte `0xAA` for outbound frames. Response headers are `0xBB` and legacy `0xCC`.

### BLE v1

```text
AA | LEN | CMD | PAYLOAD... | SUM_HI | SUM_LO
```

Length is payload length plus the command byte. Checksum is the unsigned-byte sum of the frame prefix plus seed `0x2019`; the two checksum bytes are appended high then low. This is the format documented by the older mEDIFIER captures.

### BLE v2

```text
AA | APP | CMD | LEN_HI | LEN_LO | PAYLOAD... | CHECKSUM
```

Default `APP` is decimal 236 (`0xEC`); a separate app code `0xED` is used by a small set of special operations. Length is 16-bit big-endian. Checksum is the low byte of the unsigned-byte sum of all preceding bytes. If the negotiated encryption mode is `Encryption10`, payload bytes are XORed with `0x0A`; otherwise they pass unchanged. Note: source expression sets frame `length` to payload length; verify protocol-level length semantics from captures before implementing a new codec.

Writes larger than negotiated buffer are chunked by the BLE manager. GATT notification frames may be split/concatenated; both apps have accumulation and sub-packet paths. This is a stream-framing problem, not one notification equals one command.

Source anchors: [Connect CommandBean](file:///tmp/edifier-rev/connect/decompiled/sources/com/edifier/product_connect/command/CommandBean.java), [Connect CommandManager checksum/encryption](file:///tmp/edifier-rev/connect/decompiled/sources/com/edifier/product_connect/command/CommandManager.java), [ConneX CommandManager](file:///tmp/edifier-rev/connex/sources/com/edifier/lib_connect/manager/CommandManager.java).

## Connection-time discovery and status polling

The apps do not blindly show every command in the library. They first query `0xD8` (device function/capability response) and parse it as `D8Bean`; feature bits determine which later queries, controls, and screens are enabled. In Connect, `ConnectManager.appReceiveDataListener` parses the `0xD8` reply, stores the capability bean and initiates follow-up work. ConneX's `BaseEasyCmdFragment.sendAllCmd()` starts with the same `0xD8` query; `DeviceFragment` then adds battery, ANC and EQ queries as capabilities permit.

Common status requests discovered in the UI/lifecycle paths:

| Query | Index | What the app uses it for |
| --- | ---: | --- |
| Device capabilities (`D8`) | `0xD8` / 216 | Feature discovery; model-specific UI/query gating |
| Battery | `0xD0` / 208 | Battery/status. TWS products may instead use `0xF2` |
| Version | `0xC6` / 198 | Firmware/device version |
| Bluetooth MAC/address | `0xC8` / 200 | Device identity/settings page |
| Device name | `0xC9` / 201 | Read advertised name |
| Device state | `0xF2` / 242 | State payload for supported products; special TWS paths |
| A2DP/playback state | `0xC3` / 195 | Classic audio connection/playback status |
| ANC/noise state | `0xCC` / 204 (legacy query); `0xC1` / 193 in v2 set/query family | Current noise mode / ANC state |
| EQ | `0xD5` / 213 | Current sound preset/EQ |
| Game mode | `0x08` / 8 | Low-latency state |
| Timed shutdown | `0xD3` / 211 | Timer minutes/state |
| No-audio auto-shutdown | `0xD7` / 215 | Idle auto-off setting |
| Wear detection | `0xFB` / 251 query, `0xFC` / 252 set | Wear-detection state (model dependent) |
| Touch sensitivity | `0xFD` / 253 | Touch sensitivity/current setting |
| Touch/control mappings | `0xF0` / 240 with a mapping index | Query a particular control mapping |

Connect HomeViewModel builds an ordered follow-up list from capability flags: battery or TWS state, ANC state when supported, A2DP, hearing protection, EQ, speaker volume, timing/calibration, light state, and game state. Settings screens separately request version, wear/game/tip/tap/timer state when their model skin exposes those settings. ConneX asks for support functions first, then queries supported battery/ANC/EQ and obtains additional device functions in `DeviceFragment`.

The `0xD8` payload exposes (among others) support for ANC, battery, query/set name, MAC, version, A2DP disconnect, repair, timer, no-input auto-off, EQ count/type, tap/control-setting profile, game mode, hearing protection, volume, TWS/box BLE, and newer health/light/audio capabilities. Exact byte offsets are implemented across `D8Bean` and nested beans; the catalog is product/firmware-sensitive. Do not assume every bit is valid for W820 Plus without a real response frame.

### Connect `D8` response byte layout recovered from `D8Bean`

The command response payload is parsed positionally (index below is zero-based). Missing trailing bytes leave their feature group at its default/false state. This is the Connect 8.4.28 parser layout:

| Payload byte | Parsed meaning |
| ---: | --- |
| 0 | `B0`: ANC mode/value and TWS/peer/channel flags |
| 1 | Battery capability/value field (`0` means battery display unsupported) |
| 2 | Query Bluetooth name supported |
| 3 | Set-name mode; maps to maximum UTF-8 byte length 24/30/29/35 depending on value |
| 4 | Query MAC supported |
| 5 | Query firmware/version supported |
| 6 | A2DP disconnect supported |
| 7 | Bluetooth re-pair supported |
| 8 | Auto shutdown without audio supported |
| 9 | Timed shutdown support/current timer byte |
| 10 | Device power-off supported |
| 11 | Show manual/help entry |
| 12 (`B12`) | Android SPP/BLE OTA, iOS SPP/BLE OTA, ShareMe, touch reset, step count, voice switch |
| 13 | EQ capability/count byte |
| 14 | UI skin/profile value |
| 15 (`B15`) | Shake gesture capability bits |
| 16 | Tap/control-settings profile selector (`b16Tap`); selects mapping schema/UI |
| 17 (`B17`) | LED, beep switch, in-ear detection, touch sensitivity, beep volume, reset, game mode, box battery |
| 18 (`B18`) | Fit detection, LHDC/LDAC, earmuff, wind/leak detection, clear pairing, light color |
| 19 (`B19`) | Input source, volume, ambient-light timer, music info, pressure, touch, ambient light |
| 20 (`B20`) | Sound-space, prompt tone, Hi-Res, buttons, hearing protection, time calibration/recovery, drag gestures |
| 21 (`B21`) | Time calibration, denoise mode, study mode, smart light, beep, fast charge, data upload, night light |
| 22 (`B22`) | HD audio codec, sound-pickup light, healthy tips, Bluetooth pair records, ultra-audio resolution |
| 23 (`B23`) | Saving mode, microphone, heart-rate check, adaptive/setting sound channel, line Hi-Res |
| 24 | Not assigned by this parser version |
| 25 (`B25`) | Dongle, headset state, bass, blood-oxygen check, text, EQ tips, smart no-take-off, pixel screen |
| 26 (`B26`) | HWA audio, translation, box BLE, time calibration, headset recording, power-on volume memory, TF, pickup-mic settings |
| 27 (`B27`) | Linkage-power switch bit |

Nested `Bxx` classes extract feature bits from their byte. The scalar values in bytes 1, 3, 9, 13, 14 and 16 are not boolean bitsets. In particular, `b16Tap` is a selector, not “tap enabled”; Connect uses it to build the available control-assignment tree.

## User-facing W820NB Plus controls identified in Connect

Connect recognizes Plus by search ID `0000a500-...` and chooses a dedicated `ac_tap_w820nb_plus` layout for its tap/control page. The control-setting path uses:

- `0xF0` (`tap_query`) with a `combineIndex` to read one mapping;
- `0xF1` (`tap_set`) with `[combineIndex, parentValue, (optional childValue)]` to set it;
- the mapping values/options are assembled from the `D8Bean` tap profile and product-config data, not hardcoded as one universal mapping.

The supplied Connect 8.4.28 APK decompilation further resolves this W820NB Plus profile: `D8Bean.getB16Tap()` value `18` selects profile 18; the UI builds left single/long controls at mapping indices 1/3 and right single/long at 2/4. The normal action IDs are 1 play/pause, 2 volume up/down (side-specific), 3 previous/next track (side-specific), 4 voice assistant, and 5 game/sound mode. ID 6 is a multi-select noise-mode cycle with high ANC, medium ANC, ambient, normal, and wind-reduction choices; Connect enforces selection of two or three choices. F0 replies are decoded as `[mappingIndex, selectedValue, (optional child/multi-select value)]`; F1 writes are `[mappingIndex, selectedValue]` or `[mappingIndex, selectedMask, childValue]`. This is APK-derived behavior, not a physical-device capture. Keep the controls gated on the exact D8 profile 18 and treat successful reads after writes as DeviceDeck-side confirmation; actual W820NB Plus firmware behavior still needs hardware validation.

The separate generalized touch-sensitivity setting uses `0xFD` query and `0xFE` set. In v2 the setter includes an index and value (and optional extra parameter); legacy v1 omits the index. These are distinct from the `0xF0/0xF1` action mapping commands.

Other product-level screens supported in the same app/library include ANC, sound effects/EQ, game mode, scheduled power-off, wear detection, voice prompts, device name, LDAC/LHDC, firmware/OTA, and many speaker/earbud-only functions. Their presence in source is not evidence that each is enabled or supported on W820NB Plus; `D8Bean` and the product profile gate them.

## Core command/API inventory useful for W820 development

| Function | Query / set | Payload known from source |
| --- | --- | --- |
| Capabilities | `D8` query | Empty query payload; response parsed into capability flags |
| Battery | `D0` query | Empty query payload |
| Firmware/version | `C6` query | Empty query payload |
| MAC | `C8` query | Empty query payload |
| Name | `C9` query / `CA` set | UTF-8 bytes for set; model may impose a byte limit |
| ANC | `CC` / `0xCC` query; `C1` / `0xC1` set (Connect also builds v2 control/read requests on `C1`) | v2 `[ancIndex, value, (optional progress)]`; v1 `[value, (optional progress)]` |
| EQ | `D5` query / `C4` set | Set starts with preset index, optionally followed by model EQ-calibration bytes |
| Custom EQ | `43` query, `44` set/single-band operations, `45` reset, `46` custom table transfer, `47` name | Format varies by product profile; not assume W820 Plus supports it |
| Game mode | `08` query / `09` set | Set generally carries mode byte; `09` supports optional scene/menu bytes |
| ANC gesture/button map | `F0` query / `F1` set | `[combineIndex]`; set `[combineIndex, parentValues..., optionalChild]` |
| Touch sensitivity | `FD` query / `FE` set | v2 set `[index, value, optional extra]`; legacy payload differs |
| Timed shutdown | `D3` query / `D1` set / `D2` cancel | Set minutes as two bytes big-endian |
| Auto-off without audio | `D7` query / `D6` set | Set one byte |
| Wear detection | `FB` query / `FC` set | Feature/model dependent |
| Beep/prompt tone switch | `F9` query / `FA` set | `F9` response byte 0 and `FA` payload byte 0 are boolean 0/1; Connect gates the `tws_prompt` setting on D8 B17 bit 1. This switch is separate from voice prompt level (`05`/`06`). |
| Voice prompt level | `05` query / `06` set | Set one level/progress byte |
| Power off / repair / disconnect / reset | `CE` / `CF` / `CD` / `07` | These are state-changing commands; reset/power operations should not be sent casually |
| OTA | `10` ready / `11` start-master / `12` start-sub and transfer commands | Firmware update has a separate multi-stage/chunk protocol; do not implement from index alone |

The CSV contains the full named enum, including commands unrelated to these headphones: lighting, heart rate, blood oxygen, FM, sport, translation, box controls, dongle, display/pixel screen, microphone, and speaker features. Their presence means the SDK is shared across the Edifier catalog, not that the W820 hardware implements them.

## Connect vs ConneX

- The command architecture is strikingly similar: same `0xAA` framing, v1/v2 CRC/encryption model, same prominent command IDs, D8 capability gate, local product database and command log. ConneX's command enum has 179 entries versus Connect's 187; differences are mostly product-generation/library surface, not a distinct standard API.
- For matching command names, the notable index mismatch in these supplied builds is `denoise_mode_set`: Connect `0x83`, ConneX `0x80`. Connect also has two `tips_eq_*` indices absent from the ConneX enum. Do not build one cross-app table by blindly copying every index across libraries; preserve per-protocol/version mappings.
- Connect has the explicit W820NB Plus product UUID and dedicated Plus touch-control layout in this supplied build. That is the clearest static proof of Plus-specific UI behavior.
- ConneX has support for W820 family products in its catalog/library and a dynamic feature page. The shown ConneX build's exact product GATT profile is database-backed; the supplied APK alone did not yield a plain-text W820NB Plus service/read/write UUID tuple.
- Both save command traffic into a Room table with product UUID, header, appCode, command index, payload, checksum, raw frame, direction and timestamp. Connect's table can be useful for a future runtime data export, but static decompilation does not expose the user's runtime DB.
- App versions and behavior can differ from the 4PDA thread snapshots; this report is tied to the supplied binaries' version metadata.

## Network/backend and non-headphone APIs

The product catalog is persisted locally as `ProductTable` and includes the search UUID, GATT service/read/write UUIDs, SPP UUID, product name/model, firmware metadata and feature/product presentation fields. This explains why the decompiled static constants do not contain every service UUID. Connect refreshes this catalog from `https://admin-app.edifier.com/edifier_provider/public/products?subjection=edifier`, passing `UpdateTime` and `OS` headers and caching the JSON in preferences/Room. This endpoint distributes product/config data; live headphone control frames go over Bluetooth. The supplied builds also contain substantial web/account, analytics, translation/audio, media, OTA and support functionality. Those are separate app/backend APIs, not required for the core W820 control path. Firmware update is not an ordinary command and should be analyzed independently before use.

## Exact source files to continue from

- Connect command enum: `com/edifier/product_connect/command/CommandIndex.java`
- Connect command construction / parser / CRC: `com/edifier/product_connect/command/CommandManager.java`, `CommandBean.java`
- Connect Bluetooth/GATT: `com/edifier/product_connect/connect/ble/BLEManager.java`, `ConnectManager.java`
- Connect capability map/parser: `com/edifier/product_connect/command/bean/base/D8Bean.java`, `CommandParserHelper.java`
- Connect Plus layout selection and controls: `page/common/controlsetting/ControlSettingAc.java`, `page/common/controlsetting/viewmodel/ControlSettingViewModel.java`, `res/layout/ac_tap_w820nb_plus.xml`
- Connect startup queries: `page/common/main/viewmodel/HomeViewModel.java`, `page/common/settings/viewmodel/SettingViewModel.java`
- ConneX command enum: `com/edifier/lib_connect/constant/CommandIndex.java`
- ConneX command construction / Bluetooth: `com/edifier/lib_connect/manager/CommandManager.java`, `connect/ble/BLEManager.java`
- ConneX device query orchestration / feature screen: `com/edifier/lib_common/base/BaseEasyCmdFragment.java`, `com/edifier/connex/skin_def/device/DeviceFragment.java`

Decompiled source roots are currently under `/tmp/edifier-rev/connect/decompiled` and `/tmp/edifier-rev/connex`; the original APK/APKS and RAR archives remain in the project folder. JADX decompilation is for analysis and is not the original source code.
