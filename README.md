# Starknet Source Verification Evidence

This repository specifies a portable evidence format for exact Cairo source
verification on Starknet.

The format records one narrow claim:

> A named verifier rebuilt a precisely identified Cairo source bundle under a
> precisely identified policy and toolchain, produced a canonical Starknet
> class hash, and compared it with authoritative finalized declaration
> evidence.

An `exact_match` record does not mean that a contract is secure, audited,
endorsed, currently used by an address, or proven equivalent from Sierra to
CASM. Source publication rights are also separate from verification status.

The receipt subject is a Starknet class hash, not a contract address. The same
class hash may be referenced by many contracts. Reuse on another chain requires
independent declaration evidence for that chain.

## Status

`v0.1-draft` is an implementation-neutral draft. Starkscan is the first
implementation, but no Starkscan host, database, controller protocol, or
product API is normative here.

The Verifier Alliance mapping is proposed and has not been reviewed or endorsed
by the Verifier Alliance.

## Validate

The validator uses only Python 3's standard library:

```bash
./tools/validate-fixtures
```

It checks every valid fixture, confirms every invalid fixture fails closed,
recomputes deterministic evidence identities, and rejects duplicate JSON keys.
Canonical identity and source-tree hashing are defined normatively in
[`docs/canonicalization.md`](docs/canonicalization.md).

## Layout

```text
schemas/       JSON Schemas
fixtures/      synthetic positive and negative conformance vectors
docs/          normative canonicalization, lifecycle, and security rules
mappings/      mappings to related ecosystem formats
tools/         deterministic fixture validator
```

## Claim Boundaries

This specification does not create a public source-upload or compiler service.
It does not authorize source redistribution, import external verification
evidence, promote a badge, or provide Verifier Alliance membership or
compatibility.
