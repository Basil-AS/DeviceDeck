# Device support matrix

Status meanings: **Not started** = no DeviceDeck implementation; **Researching** = source/app evidence being examined; **Protocol only** = reusable frame codec exists but there is no platform discovery/UI integration; **Implemented** = app code exists but hardware validation is incomplete; **Verified** = tested on the named OS, device revision, and firmware. “Cataloged” alone never means verified.

| Device | Android | Windows | macOS | Planned controls / research status |
|---|---|---|---|---|
| Edifier W820NB Plus | Researching | Not started | Implemented, unverified | macOS alpha.4 adds D8 capability decoding, BLE discovery/raw capture, battery, ANC/transparency, an informational capability report, and capability-gated game mode (0x08 query / 0x09 write/readback). BLE UUIDs are from a community source profile. No physical headset round-trip verification yet. Android has a merged frame codec foundation only; no GATT transport or Edifier UI. EQ, prompts, gestures and remaining controls need implementation. |
| realme Buds Air 7 | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC modes, game/high-quality/spatial audio, EQ, multipoint, gestures, wear and find mappings. DeviceDeck hardware round-trip verification is outstanding. |
| realme Buds Air 7 Pro | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC/wind, game/high-quality/spatial audio, EQ, gestures, wear and find mappings. The Windows/Linux known-issues page now warns that spatial audio may be silent (DeviceDeck-Android-Windows alpha.2). Multipoint is not in this profile. Verify controls on named hardware/firmware. |
| Baseus EP10 Pro | Researching | Not started | Implemented, unverified | macOS now opens vendor-app RFCOMM/SPP (`00001101-0000-1000-8000-00805F9B34FB`), reads left/right battery (`BA02`) and noise mode (`BA33`), and writes Off/ANC/Transparency (`BA34`) with ACK plus state readback. ANC uses the vendor app's EP10 Pro type 101. Gesture IDs exist but wire mapping is unresolved; no physical headset validation yet. Android and Windows remain unimplemented. |
| Baseus Bowie 30 | Researching | Not started | Implemented, unverified | macOS DeviceDeck routes the exact Bowie 30 name through Baseus Intelligent EU GATT UUIDs; reads battery/noise state and supports Off, Commute, Indoor, Outdoor, and Transparency with ACK plus readback. No hardware round-trip yet. Android and Windows remain unimplemented. |
| Baseus Bowie MF1 | Researching | Not started | Not started | Vendor app selects classic RFCOMM/SPP; the live BLE dictionary also lists the shared BLE UUID profile. Gesture choices exist, but wire mapping is unresolved. No explicit noise-mode selector was found. DeviceDeck has no MF1 app controls yet. |
| Baseus Blade 2 | Researching | Protocol only | Protocol only | The app-layer read/write codec, screen presets/text, timers, telemetry parser and shared GATT UUID profile are documented in [`apps/baseus`](apps/baseus). The Baseus app BLE dictionary also catalogs all 149 EU model entries. Device discovery/session handling, platform UI, and hardware round-trip remain open; Android integration is not started. |

## Evidence required to mark a feature verified

- Exact marketed model, hardware revision, regional variant, and firmware.
- OS version, Bluetooth adapter/phone, and app build.
- Read trace before and after a change; command/response decoded with confidence noted.
- Reconnect and persistence behavior.
- Whether the official app/system service must release an exclusive control connection.

For inherited realme profiles, listed controls describe mappings in the existing Android/Windows code, not verified results from a DeviceDeck build. Other feature names are investigation targets, not claims that every model supports every item.

## Current static evidence

Baseus Intelligent 2.18.0 includes model-specific UI and command paths. EP10 Pro and Bowie MF1 select RFCOMM/SPP in the observed app flow; Bowie 30 and Blade 2 use BLE, with Blade 2 on a separate power-device flow. A live regional BLE dictionary now provides UUIDs for 149 models, including all four requested Baseus models. DeviceDeck has initial EP10 Pro SPP battery/noise support and Bowie 30 BLE battery/noise plus all three vendor ANC environments on macOS; neither path has hardware validation. The Baseus protocol-reference checkout includes an EU catalog of 149 model/GATT profiles and a Blade 2-140 app-layer codec/parser, but Blade 2 still has no DeviceDeck discovery/session/UI. MF1 controls, Blade 2 telemetry and Android/Windows integration remain open. See [`research/baseus-intelligent-findings.md`](research/baseus-intelligent-findings.md), [`research/baseus-ble-config-catalog.csv`](research/baseus-ble-config-catalog.csv) and [`apps/baseus/docs/protocol/blade-2-mobile-power.md`](apps/baseus/docs/protocol/blade-2-mobile-power.md). The current realme Link base APK decompilation has not yet yielded explicit Air 7/Air 7 Pro names; continue checking splits, native/assets data, and catalog services. See [`research/RESEARCH_STATUS.md`](research/RESEARCH_STATUS.md).
