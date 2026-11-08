# DeviceDeck

Open-source, multi-platform control center for BLE devices: earbuds, headphones, speakers, and battery packs. DeviceDeck aims to expose each device's real capabilities through a lightweight, consistent interface.

## Applications

| Platform | Repository | Upstream |
|---|---|---|
| Android + Windows + Linux | [`apps/android-windows`](apps/android-windows) | [spizganed/QuickBuds](https://github.com/spizganed/QuickBuds) |
| macOS | [`apps/macos`](apps/macos) | [abaichuan1017/HeyBuds](https://github.com/abaichuan1017/HeyBuds) |
| Protocol research and device inventory | [`research/`](research/) | DeviceDeck-maintained |

App repositories are public forks. Keep upstream history intact and isolate DeviceDeck changes in small, reviewable commits so upstream changes can be pulled regularly. See [docs/UPSTREAMS.md](docs/UPSTREAMS.md).

## Pre-release builds

- [Android APK/AAB 0.1.0-alpha.3](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/android-v0.1.0-alpha.3)
- [Windows/Linux 0.1.0-alpha.3](https://github.com/Basil-AS/DeviceDeck-Android-Windows/releases/tag/desktop-v0.1.0-alpha.3)
- [macOS DMG 0.7.0-alpha.5](https://github.com/Basil-AS/DeviceDeck-macOS/releases/tag/macos-v0.7.0-alpha.5)

These are alpha builds. Android alpha.3 includes Blade 2 telemetry/display controls, and macOS alpha.5 adds Bowie MF1 battery readings alongside EP10 Pro SPP and Bowie 30 BLE controls. Android/Windows/Linux EP10 Pro and MF1 additions are in the upcoming alpha.4 release branch. Device paths remain unverified on physical hardware, and several Edifier/realme/Baseus functions are still missing. The macOS DMG is ad-hoc signed and not notarized. See the [support matrix](SUPPORT_MATRIX.md) for platform and device status.

DeviceDeck is one cross-platform product with per-platform builds. Baseus support lives inside the same DeviceDeck apps and release channels. [`research/baseus-reference`](research/baseus-reference) is a research checkout retained as protocol/source reference; it is not a separate product or release target. Its catalog of 149 BLE model profiles informs the shared device catalog, and its Blade 2-140 codec has an initial Android telemetry/display implementation. Broader port controls, Windows/macOS powerbank sessions, and physical-device verification remain open.

## Initial device scope

- Edifier W820NB Plus
- realme Buds Air 7 and Buds Air 7 Pro (the requested “Buds 7 / 7 Pro”; confirm exact model names and regional variants)
- Baseus EP10 Pro
- Baseus Bowie 30
- Baseus Bowie MF1
- Baseus Blade 2

See [device support matrix](SUPPORT_MATRIX.md) and [roadmap](docs/ROADMAP.md).

## Repository map

See [docs/PROJECT_MAP.md](docs/PROJECT_MAP.md) for where app code, protocol research, local-only inputs, assets, planning, and release workflow live.

“Support” is tracked per feature and per platform. A model catalog entry is not proof that its commands work. Features are marked verified only after read/write round-trip checks on hardware and firmware are recorded.

## Research and local inputs

Reverse-engineering notes and command index for Edifier are in [`research/edifier/`](research/edifier/). Original APKs, source archives, and complete forum captures are kept locally under `research/local-evidence/` and are intentionally excluded from Git. Do not publish vendor APKs, proprietary artwork, or complete copied forum posts. Record derived findings, command traces, and citations instead.

## Development principles

1. Discover a device and its actual capabilities before rendering controls.
2. Separate transports, vendor protocol codecs, device profiles, and UI.
3. Keep platform Bluetooth implementations native; share protocol documentation and test vectors where possible.
4. Never send unknown or destructive commands automatically. Firmware update, factory reset, and raw command consoles need explicit safeguards.
5. Preserve upstream licenses and notices in each app fork; review third-party assets before reuse.

## Status

Public alpha builds are available, but device support remains in progress. No target device is marked fully supported yet. See [SUPPORT_MATRIX.md](SUPPORT_MATRIX.md).

## Licensing

The umbrella repository's own documentation, device-profile data, and original assets are dedicated under CC0-1.0. This does not relicense the app submodules or vendor materials; each retains its own license and notices.
