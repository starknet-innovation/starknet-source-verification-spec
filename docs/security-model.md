# Security Model

## Protected Claim

The format protects the integrity and portability of evidence that a specific
Cairo source bundle rebuilt to a specific Starknet class hash under a stated
policy. It does not make the compiler, source, declaration provider, verifier,
or contract trustworthy by itself.

## Trust Boundaries

A production verifier SHOULD separate:

- untrusted source intake;
- an isolated, bounded, network-disabled build environment;
- authoritative finalized declaration resolution;
- independent class-hash recomputation;
- receipt admission and lifecycle authority; and
- public evidence storage.

The build environment MUST NOT hold receipt-signing material, production
database credentials, badge-write credentials, or source-provider credentials.
Its output is evidence, never its own authorization to publish a badge.

## Threats and Required Controls

### Malicious JSON

Parsers MUST reject duplicate keys, invalid UTF-8, non-NFC strings, unsupported
fields, invalid enums, unsafe integers, floats, excessive nesting, and records
larger than an implementation's documented bound. A recommended envelope limit
is 1 MiB and a recommended nesting limit is 32.

### Malicious Source Archives

Intake MUST reject path traversal, absolute paths, duplicate normalized paths,
links, special files, sparse files, archive bombs, and material outside
documented file/count/size limits. Archive inspection and extraction MUST use
the same normalized path policy.

### Mutable Build Inputs

Exact verification MUST bind `Scarb.toml`, `Scarb.lock`, compiler binaries, all
dependency materials, selectors, profile, and build environment. Network
resolution during a qualifying build is forbidden. Missing or undeclared
materials fail closed.

### Compromised Builder

A builder may lie about its output. A separate trusted component SHOULD
recompute the produced Sierra class hash and compare it with independently
resolved finalized declaration evidence before admitting an exact receipt.

### False Finality or Chain Confusion

Submission chain, declaration chain, transaction hash, block number, and
response digest are bound independently. V0 requires `accepted_on_l1`.
Evidence from one chain MUST NOT establish declaration on another chain.

### Secret and Private-Source Leakage

Portable records MUST NOT contain credentials, cookies, authenticated provider
URLs, private source, request bodies, internal host/database/actor identifiers,
or raw build logs. Use content digests and visibility/retention/redistribution
classifications. Public source publication requires separate authorization.

### Revocation Abuse

Structural validity is not lifecycle authority. Consumers MUST authenticate
issuers out of band, preserve all transitions, reject cycles, and never let a
replacement inherit validity without independent validation.

## Extension Safety

Extensions use reverse-domain names. Unknown extensions MUST NOT change the
core exact-match decision. A consumer MAY ignore an unknown extension, but it
MUST preserve it when verifying `evidenceDigest`.

A future Sierra-to-CASM proof belongs in an extension. It provides a
complementary guarantee and MUST NOT be implied by a core V0 exact source match.

