# Architecture and repository layout

```text
DeviceDeck/                         # Public umbrella: roadmap, matrix, research index
├── apps/android-windows/            # QuickBuds fork: Android / Windows / Linux
├── apps/macos/                      # HeyBuds fork: macOS
├── research/edifier/                # Static analysis report and command index
├── research/local-evidence/          # Local-only APKs, zips, raw forum captures (gitignored)
├── assets/icons/                    # Original app/brand icon work with provenance
└── assets/device-images/             # Original/licensed device illustrations
```

Each app remains an independent Git repository and an upstream fork. The umbrella repo records the exact app commit as a submodule so a checkout is reproducible. Protocol documentation lives centrally; Bluetooth APIs stay platform-native. Once the packet formats are stable, a small language-neutral profile schema and JSON golden traces can be shared across apps.

## Device profile layers

1. **Transport** — OS Bluetooth discovery, GATT/RFCOMM connection and notification lifecycle.
2. **Vendor protocol** — framing, checksums, command IDs, capability negotiation, decoding.
3. **Model profile** — model identity and supported capabilities/allowed value ranges.
4. **Normalized state/actions** — battery, ANC, EQ, latency, gestures, etc.
5. **UI** — only render controls confirmed by the active profile and current capability response.

A device may expose different features by firmware, region, transport, and connected host. Profiles must describe these differences rather than hard-code a universal feature set.
