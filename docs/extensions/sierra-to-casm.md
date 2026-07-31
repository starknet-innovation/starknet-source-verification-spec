# Optional Sierra-to-CASM Evidence Extension

**Status: proposed, non-normative.** This document describes an optional
extension for a verifier that can prove that a rebuilt Starknet Sierra artifact
produces the CASM class hash used by Starknet. It is not part of the core V0
exact-source decision and has not been reviewed or endorsed by the Verifier
Alliance.

The reference implementation under evaluation is
[`sierra-to-casm-compilation-proof`](https://github.com/starknet-innovation/sierra-to-casm-compilation-proof).
The extension can be adopted by other Starknet verifiers once its input/output
interface and proof checker are stable.

## What the extension adds

Core V0 evidence answers:

> Did the identified source bundle rebuild to the declared Sierra class hash?

This extension adds a separate question:

> Does that Sierra artifact compile to the compiled-class hash Starknet uses?

The second result is complementary. A missing or invalid extension record MUST
NOT turn a valid core source match into a mismatch, and a Sierra-to-CASM proof
MUST NOT by itself promote a source bundle to an exact source match.

## Proposed bounded record

An implementation MAY attach an extension under a reverse-domain key such as
`org.starknet.sierra-to-casm.proof-v0`:

```json
{
  "extension": "org.starknet.sierra-to-casm.proof-v0",
  "status": "verified",
  "sierraArtifactDigest": "sha256:<64 lowercase hex characters>",
  "expectedCompiledClassHash": "0x<canonical felt>",
  "proofDigest": "sha256:<64 lowercase hex characters>",
  "proofSystem": {
    "name": "<proof system name>",
    "version": "<pinned version>",
    "checkerDigest": "sha256:<64 lowercase hex characters>"
  }
}
```

The portable record SHOULD contain digests and bounded metadata, not an
unbounded proof blob. A separate artifact store MAY retain the proof itself,
subject to the same source-retention and redistribution rules as the core
record.

## Required bindings

The extension MUST bind to:

1. the core `evidenceDigest` it supplements;
2. the exact Sierra artifact digest produced by the source build;
3. the expected compiled-class hash and Starknet chain;
4. the proof system and checker versions; and
5. the proof digest or an equivalent content-addressed reference.

The proof checker MUST reject a different Sierra artifact, chain, compiled
class hash, or checker version. The same inputs and pinned checker MUST produce
the same proof result and digest.

## Failure and compatibility rules

Consumers that understand the extension MUST treat these states separately:

- `verified`: the proof was checked successfully;
- `unavailable`: no proof was produced or the checker was not installed;
- `rejected`: a proof was supplied but failed validation; and
- `unsupported`: the consumer does not implement this extension.

`unavailable` and `unsupported` preserve the core source-verification result.
`rejected` is important evidence and SHOULD be surfaced, but it does not change
the core result unless the verifier's published policy explicitly requires the
extension for a different, stronger tier.

## Starkscan integration checklist

- [ ] Agree on the smallest stable proof input/output interface with the proof
      repository maintainers.
- [ ] Attach the extension only after Starkscan's existing source build has
      produced the Sierra artifact.
- [ ] Keep proof generation outside badge admission and receipt signing.
- [ ] Add exact-match, wrong-Sierra, wrong-class-hash, corrupted-proof, missing
      checker, and deterministic-replay fixtures.
- [ ] Record only bounded metadata and a digest in portable evidence.
- [ ] Document the proof toolchain and checker trust boundary.
- [ ] Propose the extension to the Alliance only after the interface and
      vectors are stable.

This extension is intentionally separate from the core schema so that source
verification remains useful while the Sierra-to-CASM proof system matures.
