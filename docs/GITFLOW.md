# GitFlow and release process

## Branches

- `main` contains public, released versions.
- `develop` is the integration branch.
- `feature/<topic>` branches start at `develop` and merge by pull request into `develop`.
- `release/<version>` branches start at `develop`; version and release notes are finalized there, then the branch is merged to `main` by pull request and tagged. Merge `main` back to `develop` after release.
- `hotfix/<topic>` starts at `main`; merge it by PR to `main`, then back to `develop`.

All three repositories have `develop` branches. Never push feature commits directly to `main` or `develop`. The umbrella repository records exact app submodule commits; update its submodule pointers through a separate DeviceDeck PR after the app PR is merged.

## Commit attribution

The authenticated GitHub account is the committer. Per the project owner's request, DeviceDeck-authored commits use this trailer:

```text
Co-authored-by: OpenAI Codex <codex@openai.com>
```

Do not attribute upstream authorship or rewrite upstream commits.

## Versioning and releases

- DeviceDeck coordination/docs uses SemVer tags `vMAJOR.MINOR.PATCH`.
- Each app preserves its own runtime versioning/build-number rules and release-signing/update-feed rules. Never reuse upstream signing keys, update feeds, package identifiers, or appcast URLs for a fork.
- Release PRs must link CI artifacts and list per-device/per-OS support evidence. Only publish a release after build/package checks pass and the release artifact can be installed. Hardware limitations and unverified controls must be explicit.
- Secrets (Android keystore, macOS notarization credentials, Sparkle key) belong only in repository Actions secrets or local keychains; never commit them.
