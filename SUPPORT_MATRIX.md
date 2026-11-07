# Device support matrix

Status meanings: **Not started** = no DeviceDeck implementation; **Researching** = source/app evidence being examined; **Implemented** = code exists but hardware validation is incomplete; **Verified** = tested on the named OS, device revision, and firmware. “Cataloged” alone never means verified.

| Device | Android | Windows | macOS | Planned controls / research status |
|---|---|---|---|---|
| Edifier W820NB Plus | Researching | Not started | Implemented, unverified | macOS has BLE discovery, raw D8 capture, battery, ANC/transparency, and now decodes the documented D8 capability report for the device-info page. These reported capabilities are informational; EQ, game mode, prompts, gestures and other controls still need command implementation and hardware round-trip validation. BLE UUIDs are from a community source profile; no physical headset verification yet. |
| realme Buds Air 7 | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC modes, game/high-quality/spatial audio, EQ, multipoint, gestures, wear and find mappings. DeviceDeck hardware round-trip verification is outstanding. |
| realme Buds Air 7 Pro | Implemented, unverified | Implemented, unverified | Researching | Android/Windows inherit a model profile with battery, ANC/wind, game/high-quality/spatial audio, EQ, gestures, wear and find mappings. Multipoint is not in this profile. Verify each control and platform on named hardware/firmware. |
| Baseus EP10 Pro | Researching | Not started | Not started | Extract model/profile and command capabilities from Baseus Intelligent; hardware validation required |
| Baseus Bowie 30 | Researching | Not started | Not started | Extract model/profile and command capabilities from Baseus Intelligent; hardware validation required |
| Baseus Bowie MF1 | Researching | Not started | Not started | Extract model/profile and command capabilities from Baseus Intelligent; hardware validation required |
| Baseus Blade 2 | Researching | Not started | Not started | Determine whether it exposes BLE controls/status; distinguish BLE control from USB-C/power telemetry |

## Evidence required to mark a feature verified

- Exact marketed model, hardware revision, regional variant, and firmware.
- OS version, Bluetooth adapter/phone, and app build.
- Read trace before and after a change; command/response decoded with confidence noted.
- Reconnect and persistence behavior.
- Whether the official app/system service must release an exclusive control connection.

For inherited realme profiles, listed controls describe mappings in the existing Android/Windows code, not verified results from a DeviceDeck build. Other feature names are investigation targets, not claims that every model supports every item.

## Current static evidence

Baseus Intelligent 2.18.0 contains named app-side profiles/config entries for all four requested Baseus products: `Baseus Bass EP10 Pro`, `Bowie 30`, `Baseus Bowie MF1`, and `Blade 2`. Some control/gesture resources are model-grouped. This is a research lead, not a working-feature verification. The current realme Link base APK decompilation has not yet yielded explicit Air 7/Air 7 Pro names; continue checking splits, native/assets data, and catalog services. See [`research/RESEARCH_STATUS.md`](research/RESEARCH_STATUS.md).
