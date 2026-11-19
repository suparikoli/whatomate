# ADR-067: App builds run on our own machines, not hosted CI

Status: proposed (G1, module `apps`).

## Context
- Rule E12 bans GitHub Actions and other paid GitHub features.
- iOS and macOS builds, signing and notarisation need macOS and Xcode.
- MSI builds need Windows. NSIS cross-builds from macOS or Linux are experimental.
- The home k3s cluster runs Linux pods. The Macs are short on memory.

## Decision
- iOS and macOS: build on a Mac with Xcode. Sign, notarise with `notarytool`, upload with an App Store Connect API key.
- Android and Linux desktop (AppImage, deb, rpm): build in pods on the home cluster.
- Windows: build on a Windows machine or VM. Sign with an OV/EV cert or Azure Trusted Signing.
- Each repo has a `release` script per platform. It reads every key from env vars (E10).
- Steps: bump version, build, sign, notarise, TestFlight and Play internal track, store review, staged rollout. Desktop: upload installers and a signed updater manifest.

## Alternatives
- Hosted CI (GitHub Actions, Codemagic, Bitrise). Rejected by E12 and cost.
- Cross-build everything on one Mac. Rejected: Windows signing and MSI need Windows; memory is short.

## Consequences
- The user supplies store accounts, certificates, keys and a Windows build host.
- Releases are manual and need one checklist per platform (G3).
