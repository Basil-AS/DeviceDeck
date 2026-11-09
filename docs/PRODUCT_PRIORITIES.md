# DeviceDeck product priorities

**Decision recorded: 2026-11-10**

## One product, Android first

DeviceDeck is one product with platform builds. Every brand and device profile—including Baseus—belongs inside DeviceDeck. Protocol research folders and source checkouts are engineering material only; they are not separate apps or release channels.

Android is the clear, overwhelming development priority. Put the available implementation, protocol-research, UX, and release effort into Android first. Windows, Linux, and macOS are secondary builds: maintain them and port Android features when practical, but their parity must not delay Android work or releases. Android can ship a feature without waiting for desktop parity.

When time or capacity is limited, prioritize work in this order:

1. Android device discovery, connection reliability, device-specific controls, status, recovery, and visual quality.
2. Android feature completeness for the named target devices, with evidence and clear per-feature confidence.
3. Android release quality, accessibility, performance, and usable error states.
4. Broader Android device coverage, after the named targets have useful, evidence-backed support.
5. Windows, Linux, and macOS parity for the same device profiles and controls.

This is a delivery priority, not a reason to weaken protocol correctness or hide known limitations. A feature should only be shown when its model capability is established; uncertain or hardware-unverified behavior must be labeled accordingly.

## Release and repository practice

- Keep platform builds under DeviceDeck branding and use the established GitFlow branches, pull requests, CI, tags, and release channels.
- Implement and review Android changes first when they affect shared device behavior, then bring compatible controls to Windows and macOS.
- Preserve upstream history and licenses in platform forks. Keep protocol notes, vendor-source research, and product binaries out of separate Baseus application repositories.
- Keep the support matrix and changelog aligned with each published build. Report hardware verification separately from a successful CI build.
