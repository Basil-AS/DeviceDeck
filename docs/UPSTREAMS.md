# Upstream and fork workflow

| Local path | Public fork | Upstream | Notes |
|---|---|---|---|
| `apps/android-windows` | `Basil-AS/DeviceDeck-Android-Windows` | `spizganed/QuickBuds` | Android, Windows, Linux base; GPL-3.0 upstream |
| `apps/macos` | `Basil-AS/DeviceDeck-macOS` | `abaichuan1017/HeyBuds` | macOS multi-brand base; inspect license/notices before each reuse |

Each checkout has `origin` pointing to the DeviceDeck fork and `upstream` pointing to the original project. To update a fork:

```sh
git -C apps/android-windows fetch upstream
git -C apps/android-windows log --oneline HEAD..upstream/main
git -C apps/android-windows merge upstream/main

git -C apps/macos fetch upstream
git -C apps/macos log --oneline HEAD..upstream/main
git -C apps/macos merge upstream/main
```

Review and resolve upstream changes before pushing. Avoid rewriting upstream history. Keep DeviceDeck work on named branches and submit changes upstream when they are general improvements. Keep model profiles and vendor protocol additions isolated so they remain easy to rebase.

## Licensing

The umbrella repository does not replace app licenses. Preserve each upstream's `LICENSE`, notices, and attribution. QuickBuds is GPL-3.0. HeyBuds is GPL-3.0-or-later in the current upstream source. Check exact dependency and asset licenses before copying code or images between projects. Do not commit vendor APKs, firmware, or proprietary assets.
