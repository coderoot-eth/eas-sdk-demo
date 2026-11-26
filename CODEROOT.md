# Releasing, checking and revoking through CodeRoot

This copy of the SDK is released through CodeRoot. Each version is published to npm with
provenance and then waits in the CodeRoot queue. A person approves it there with a passkey, and
CodeRoot attests its exact bytes on EAS on Base Sepolia. A build that depends on the package checks
those bytes before it installs them. A revoke in the queue makes the next check fail.

## Adopting it

Everything the release needs is one added file, [`.github/workflows/release.yml`](.github/workflows/release.yml).
The SDK's source, tests and build are upstream's. Pushing a tag `vX.Y.Z` that matches
`package.json` runs three jobs:

| job | holds | does |
|---|---|---|
| `pack` | read access only | installs with the upstream pnpm setup, lints, tests, builds, and packs the tarball once |
| `publish` | the `npm-publish` environment, the only place npm's trusted publishing accepts | publishes that exact tarball with `--provenance`, after checking its sha256 against `pack`'s. No npm token exists. |
| `submit` | an identity token that cannot publish | hands the same tarball to the CodeRoot Action (`mode: submit`, `flow: published`) |

To adopt it in another repository:

1. Copy `release.yml`.
2. Make the repository public. npm only attaches provenance from a public repository.
3. Create the `npm-publish` environment, limited to tags `v*`.
4. On npmjs.com, add a trusted publisher for the package: this repository, `release.yml`, environment
   `npm-publish`.
5. Ask CodeRoot to link the repository, the workflow and the package to your workspace. Until then
   the submit job is refused with `repository_not_authorized`.

## Checking it

A consumer checks the bytes it is about to install, never the name and version alone.
[`coderoot-eth/eas-sdk-demo-consumer`](https://github.com/coderoot-eth/eas-sdk-demo-consumer) does this
in CI before `npm ci`:

```yaml
- uses: coderoot-eth/action@4367129c8915b9f6733044cb02d5a5adf8e301b3 # v1.2.0
  with:
    mode: verify
    hash: sha512:<the package's integrity from package-lock.json, as hex>
    fail-open: "false"
    fail-on: unregistered
- uses: coderoot-eth/action@4367129c8915b9f6733044cb02d5a5adf8e301b3 # v1.2.0
  with:
    mode: check
    lockfile: package-lock.json
```

The first step requires the SDK to read `verified`. The second checks every dependency in the
lockfile and fails only on revoked or mismatched bytes. From a terminal:

```bash
npx -p @coderoot-eth/verify@0.1.1 coderoot verify --lockfile package-lock.json
```

| verdict | meaning | exit |
|---|---|--:|
| `verified` | these bytes match a release a person approved | 0 |
| `unregistered` | no approved release has these bytes | 2 in `verify`, 0 in `check` |
| `revoked` | these bytes were approved and then revoked | 1 |

## Revoking it

A person with approval rights revokes the release in the CodeRoot queue, with a passkey. CodeRoot
revokes its attestation on EAS. The next check of those bytes reads `revoked` and fails with exit 1,
wherever it runs. To recover, pin the previous approved version: its bytes still read `verified`.

## Gas

The publisher pays no gas: CodeRoot's attester writes the attestation. Measured on Base Sepolia for
2.10.0-demo.1:

| transaction | gas | fee on Base Sepolia |
|---|--:|--:|
| [attest the release](https://sepolia.basescan.org/tx/0xcdad0791fbbaebf1bdb67fca255440122e2aae645894c02d4db8e69def1f00b9) (one record) | 900,889 | 0.0000135 ETH |

Fees on Base mainnet depend on its gas price at the time.
