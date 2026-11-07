# Device support matrix

Status meanings: **Not started** = no DeviceDeck implementation; **Researching** = source/app evidence being examined; **Implemented** = code exists but hardware validation is incomplete; **Verified** = tested on the named OS, device revision, and firmware. “Cataloged” alone never means verified.

| Device | Android | Windows | macOS | Planned controls / research status |
|---|---|---|---|---|
| Edifier W820NB Plus | Researching | Not started | Researching | Battery, ANC/transparency, EQ, game mode, prompt/gesture controls, device info; use capability responses and validate each write |
| realme Buds Air 7 | Researching | Researching | Researching | Battery, ANC, EQ, spatial/high-quality/low-latency modes, multipoint, gestures, wear/find where the model exposes them |
| realme Buds Air 7 Pro | Researching | Researching | Researching | Battery, ANC/wind reduction, EQ, spatial/high-quality/low-latency modes, gestures, wear/find; validate each platform |
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

Feature names above are investigation targets, not claims that every model supports every item.

## Current static evidence

Baseus Intelligent 2.18.0 contains named app-side profiles/config entries for all four requested Baseus products: `Baseus Bass EP10 Pro`, `Bowie 30`, `Baseus Bowie MF1`, and `Blade 2`. Some control/gesture resources are model-grouped. This is a research lead, not a working-feature verification. The current realme Link base APK decompilation has not yet yielded explicit Air 7/Air 7 Pro names; continue checking splits, native/assets data, and catalog services. See [`research/RESEARCH_STATUS.md`](research/RESEARCH_STATUS.md).
