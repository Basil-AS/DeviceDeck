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
