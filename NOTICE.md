# Notice

This repository is a copy of
[ethereum-attestation-service/eas-sdk](https://github.com/ethereum-attestation-service/eas-sdk)
at commit `9ebc43fc0345679c8a521a1f497592015b9deb17` (v2.10.0), used under its MIT license. The
first commit on `main` is that commit's tree, unchanged. `LICENSE` is upstream's, unchanged.

It is an unofficial demo, released through CodeRoot to show release attestation. It is not
published, reviewed or endorsed by the Ethereum Attestation Service. The official package is
[`@ethereum-attestation-service/eas-sdk`](https://www.npmjs.com/package/@ethereum-attestation-service/eas-sdk).

## Changes from upstream

| file | change |
|---|---|
| `package.json` | `name` is `@coderoot-eth/eas-sdk-demo`; `version` is `2.10.0-demo.N`; `repository` is this repository; `description` says it is an unofficial demo copy |
| `README.md` | a disclaimer added at the top; the rest is upstream's |
| `NOTICE.md` | this file, added |
| `CODEROOT.md` | added: how this copy is released, checked and revoked, with the gas of the attestation |
| `.github/workflows/release.yml` | added: builds, publishes to npm with provenance, and submits the release to CodeRoot |

The SDK's source, its tests, its build and its dependencies are unchanged. Published tarballs differ
from upstream's because the name and version differ, so their hashes are this copy's own and never
upstream's.
