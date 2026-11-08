# Device support matrix

Status meanings: **Not started** = no DeviceDeck implementation; **Researching** = source/app evidence being examined; **Protocol only** = reusable frame codec exists but there is no platform discovery/UI integration; **Implemented** = app code exists but hardware validation is incomplete; **Verified** = tested on the named OS, device revision, and firmware. “Cataloged” alone never means verified.

| Device | Android | Windows | macOS | Planned controls / research status |
|---|---|---|---|---|
| Edifier W820NB Plus | Researching | Not started | Implemented, unverified | macOS alpha.4 adds D8 capability decoding, BLE discovery/raw capture, battery, ANC/transparency, an informational capability report, and capability-gated game mode (0x08 query / 0x09 write/readback). BLE UUIDs are from a community source profile. No physical headset round-trip verification yet. Android has a merged frame codec foundation only; no GATT transport or Edifier UI. EQ, prompts, gestures and remaining controls need implementation. |
| realme Buds Air 7 | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC modes, game/high-quality/spatial audio, EQ, multipoint, gestures, wear and find mappings. DeviceDeck hardware round-trip verification is outstanding. |
| realme Buds Air 7 Pro | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC/wind, game/high-quality/spatial audio, EQ, gestures, wear and find mappings. The Windows/Linux known-issues page now warns that spatial audio may be silent (DeviceDeck-Android-Windows alpha.2). Multipoint is not in this profile. Verify controls on named hardware/firmware. |
| Baseus EP10 Pro | Researching | Not started | Implemented, unverified | macOS now opens vendor-app RFCOMM/SPP (`00001101-0000-1000-8000-00805F9B34FB`), reads left/right battery (`BA02`) and noise mode (`BA33`), and writes Off/ANC/Transparency (`BA34`) with ACK plus state readback. ANC uses the vendor app's EP10 Pro type 101. Gesture IDs exist but wire mapping is unresolved; no physical headset validation yet. Android and Windows remain unimplemented. |
| Baseus Bowie 30 | Researching | Not started | Not started | Vendor app offers Off/Transparency/Noise Reduction and uses the same `BA33`/`BA35` queries and `BA34 mode level` write path. Target UUID config and state/readback parser are unresolved. DeviceDeck has no Baseus transport/UI yet. |
| Baseus Bowie MF1 | Researching | Not started | Not started | Vendor app selects classic RFCOMM/SPP; gesture choices exist, but their wire mapping is unresolved. No explicit noise-mode selector was found. DeviceDeck has no Baseus protocol/UI yet. |
| Baseus Blade 2 | Researching | Protocol only | Protocol only | The Baseus app's app-layer read/write codec, screen presets/text, timers and telemetry response parser now exist in [`apps/baseus`](apps/baseus). No Blade 2 discovery, GATT UUID profile, platform UI, or hardware round-trip exists yet. Android integration is also not started. |

## Evidence required to mark a feature verified

- Exact marketed model, hardware revision, regional variant, and firmware.
- OS version, Bluetooth adapter/phone, and app build.
- Read trace before and after a change; command/response decoded with confidence noted.
- Reconnect and persistence behavior.
- Whether the official app/system service must release an exclusive control connection.

For inherited realme profiles, listed controls describe mappings in the existing Android/Windows code, not verified results from a DeviceDeck build. Other feature names are investigation targets, not claims that every model supports every item.

## Current static evidence

Baseus Intelligent 2.18.0 includes model-specific UI and command paths. EP10 Pro and Bowie MF1 use RFCOMM/SPP; Bowie 30 and Blade 2 use BLE, with Blade 2 on a separate power-device flow. EP10 Pro and Bowie 30 have noise-mode queries/writes (`BA33`/`BA35`, `BA34`). DeviceDeck has implemented the initial EP10 Pro SPP battery/noise path on macOS only; it remains hardware-unverified. The Baseus desktop fork now has a tested Blade 2 app-layer codec and parser, but no device transport/discovery or UI integration. Bowie 30's BLE configuration/readback, MF1's supported operations, Blade 2 live BLE telemetry, and Android integration remain open work. See [`research/baseus-intelligent-findings.md`](research/baseus-intelligent-findings.md) and [`apps/baseus/docs/protocol/blade-2-mobile-power.md`](apps/baseus/docs/protocol/blade-2-mobile-power.md). The current realme Link base APK decompilation has not yet yielded explicit Air 7/Air 7 Pro names; continue checking splits, native/assets data, and catalog services. See [`research/RESEARCH_STATUS.md`](research/RESEARCH_STATUS.md).
