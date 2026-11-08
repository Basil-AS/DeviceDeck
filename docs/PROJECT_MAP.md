# Project map

DeviceDeck is an umbrella repository for platform apps, vendor protocol research, and shared project planning. App implementations remain in their own public GitHub repositories and are pinned here as Git submodules. The umbrella repo is the place to compare platform coverage and coordinate model-level evidence; it is not a shared cross-platform runtime.

## Top-level files and directories

| Path | Purpose |
|---|---|
| `README.md` | Project overview, current prerelease links, and entry points. |
| `SUPPORT_MATRIX.md` | Per-device and per-platform implementation/verification status. |
| `docs/ROADMAP.md` | Phased work plan, target order, and validation gate. |
| `docs/ARCHITECTURE.md` | Platform boundaries and shared protocol/profile direction. |
| `docs/UPSTREAMS.md` | Public fork/upstream remotes, sync procedure, and license notes. |
| `docs/GITFLOW.md` | Branch, PR, tag, and commit attribution workflow. |
| `docs/ASSET_POLICY.md` | App icon and device-image locations and redistribution rules. |
| `apps/android-windows/` | Android, Windows, and Linux app submodule (`Basil-AS/DeviceDeck-Android-Windows`). |
| `apps/macos/` | macOS app submodule (`Basil-AS/DeviceDeck-macOS`). |
| `assets/icons/` | Original DeviceDeck app mark and icon notes. |
| `assets/device-images/` | Original vector form-factor illustrations and a catalog mapping all seven target models to a silhouette. These are generic interface illustrations, not product renders or protocol-support claims. |
| `research/` | Evidence index, app/version hashes, model catalogs, reverse-engineering reports, and command indexes. |
| `research/local-evidence/` | User-supplied APKs, archives, forum exports, and decompilation outputs; ignored by Git and local-only. |

## Where device work goes

- Put conclusions from source/app analysis in `research/`, with source version/hash and confidence. Keep full APKs, proprietary artwork, and copied forum dumps local.
- Put implementation in the platform app that owns the Bluetooth transport. Keep a codec and model profile separated from UI; do not treat app catalog strings as proof of working commands.
- DeviceDeck is a single product. Add brands, device profiles, protocol sessions and device-specific controls to the existing per-platform DeviceDeck apps and their normal release channels; do not create brand-specific companion apps or release repos. Research forks may be used as references only.
- Update `SUPPORT_MATRIX.md` when a feature changes status. Record evidence before using “verified”: exact model/revision/firmware, OS/app, read/write/read-back, reconnect, and persistence.
- Add original artwork to `assets/`, with a nearby source/rights record for anything not created for DeviceDeck. The APK icons and product renders are not cleared for reuse.
- Update the submodule pointer only after the corresponding platform PR is merged. Use a separate umbrella PR for that gitlink change.

## Current release entry points

The umbrella README links the current alpha releases. Platform-specific build, signing, CI, and release instructions are in each submodule's `README.md`, `docs/RELEASING.md`, or workflow files. Alpha builds are not evidence that every listed model or control works.
