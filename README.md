# Firmware Release Manifest

Canonical, signed firmware release metadata with artifact integrity, hardware compatibility, staged rollout, SBOM linkage, and update eligibility.

The manifest connects firmware images, ELF/map files, symbols, SBOMs, provenance, release notes, board constraints, bootloader requirements, and Ed25519 signatures in one deterministic artifact.

## Quick start

```bash
npm install
npm test
node src/cli.js build examples/release.json manifest.json
node src/cli.js keygen release-key
node src/cli.js sign manifest.json release-key.private.pem signed.json
node src/cli.js verify signed.json release-key.public.pem examples
```

Private keys are created with owner-only file permissions where supported. Production signing should use a protected CI secret or hardware-backed signer.

## Manifest contents

- Product, semantic version, channel, creation/expiration time, and release ID.
- One or more artifacts with type, filename, media type, byte size, SHA-256, HTTPS URL, and metadata.
- Compatible board, variant, and hardware revision ranges.
- Minimum bootloader version.
- Deterministic percentage rollout and seed.
- Release notes, build provenance, and SBOM artifact linkage.
- One or more detached Ed25519 signatures.

Artifact bytes never enter the manifest; their exact size and digest do.

## Canonical signing

`canonicalize` recursively sorts object keys, preserves array order, omits undefined properties, and rejects unsupported or non-finite values. Signatures are excluded from their own signing payload. Adding or replacing a signature therefore does not invalidate other signatures, while changing any release content does.

Verification accepts a key-ID → public-key trust map and an optional signature threshold. It reports valid, invalid, and unknown keys separately. Key IDs are the first 128 bits of the SHA-256 SPKI fingerprint.

## Device eligibility

`evaluateUpdate` checks:

- board, variant, and hardware revision compatibility;
- bootloader minimum;
- upgrade/downgrade/reinstall policy;
- release channel;
- expiration;
- deterministic staged-rollout membership.

Every rejection is returned as an explicit reason. Rollout buckets hash the release seed and device ID, keeping enrollment stable across repeated checks.

## Release comparison

`compareReleases` reports version direction, channel and target changes, and added/removed/changed/unchanged artifacts with size and digest changes. This supports promotion gates and release review.

## License

MIT
