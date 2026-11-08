# Device support matrix

Status meanings: **Not started** = no DeviceDeck implementation; **Researching** = source/app evidence being examined; **Implemented** = code exists but hardware validation is incomplete; **Verified** = tested on the named OS, device revision, and firmware. “Cataloged” alone never means verified.

| Device | Android | Windows | macOS | Planned controls / research status |
|---|---|---|---|---|
| Edifier W820NB Plus | Researching | Not started | Implemented, unverified | macOS alpha.4 adds D8 capability decoding, BLE discovery/raw capture, battery, ANC/transparency, an informational capability report, and capability-gated game mode (0x08 query / 0x09 write/readback). BLE UUIDs are from a community source profile. No physical headset round-trip verification yet. Android has a merged frame codec foundation only; no GATT transport or Edifier UI. EQ, prompts, gestures and remaining controls need implementation. |
| realme Buds Air 7 | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC modes, game/high-quality/spatial audio, EQ, multipoint, gestures, wear and find mappings. DeviceDeck hardware round-trip verification is outstanding. |
| realme Buds Air 7 Pro | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC/wind, game/high-quality/spatial audio, EQ, gestures, wear and find mappings. The Windows/Linux known-issues page now warns that spatial audio may be silent (DeviceDeck-Android-Windows alpha.2). Multipoint is not in this profile. Verify controls on named hardware/firmware. |
| Baseus EP10 Pro | Researching | Not started | Not started | App has gesture/noise UI profile hints; current product-resource API ZIP is media only, with no command schema. Capture feature writes/readback and validate on hardware. |
| Baseus Bowie 30 | Researching | Not started | Not started | App lists Off/transparency/noise reduction and gesture choices; current product-resource API ZIP is media only. Capture mode/gesture writes/readback and validate on hardware. |
| Baseus Bowie MF1 | Researching | Not started | Not started | App has gesture profile hints; no target command map or explicit ANC mapping yet. Current product-resource API ZIP is media only. |
| Baseus Blade 2 | Researching | Not started | Not started | App recognizes mobile-power stats/version UI; current product-resource API ZIP is media only. Establish if telemetry/control is BLE and validate safe read-only operations first. |

## Evidence required to mark a feature verified

- Exact marketed model, hardware revision, regional variant, and firmware.
- OS version, Bluetooth adapter/phone, and app build.
- Read trace before and after a change; command/response decoded with confidence noted.
- Reconnect and persistence behavior.
- Whether the official app/system service must release an exclusive control connection.

For inherited realme profiles, listed controls describe mappings in the existing Android/Windows code, not verified results from a DeviceDeck build. Other feature names are investigation targets, not claims that every model supports every item.

## Current static evidence

Baseus Intelligent 2.18.0 contains named app-side profiles/config entries for all four requested Baseus products: `Baseus Bass EP10 Pro`, `Bowie 30`, `Baseus Bowie MF1`, and `Blade 2`. A live product-resource API check returned presentation media/tutorial ZIPs only, with no command schema. This is a research lead, not working support. The current realme Link base APK decompilation has not yet yielded explicit Air 7/Air 7 Pro names; continue checking splits, native/assets data, and catalog services. See [`research/RESEARCH_STATUS.md`](research/RESEARCH_STATUS.md).
