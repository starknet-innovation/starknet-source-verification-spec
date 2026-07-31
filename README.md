# Starknet source verification evidence

When an explorer says that a contract is verified, people should be able to
understand what was checked and repeat the check later.

This repository is an open draft for recording that evidence for Cairo and
Starknet. It is intentionally small: a verifier rebuilds a named source bundle
with a pinned toolchain, gets a canonical Sierra class hash, and compares it
with the class declared on Starknet at accepted-on-L1 finality.

The result is portable evidence, not a trust-me badge.

## What an exact match means

An `exact_match` record means that the identified source and build inputs
produced the identified Starknet class hash under the stated verification
policy.

It does not mean that the contract is safe, audited, endorsed, or currently
used by a particular address. It also does not, by itself, prove the
Sierra-to-CASM step. Source publication and redistribution rights are recorded
separately from the verification result.

The subject is a class hash rather than an address. Several contracts can use
the same class, but each chain still needs its own declaration evidence.

## Why this is separate from Starkscan

Starkscan is the first implementation, but this repository is not a Starkscan
database, API, controller protocol, or deployment package. Other Starknet
explorers and verifiers should be able to use the format without adopting
Starkscan's infrastructure.

The design is inspired in part by the [Verifier
Alliance](https://verifieralliance.org/), especially its effort to make
verification data easier to share across providers. The Alliance mapping here
is a proposal for discussion, not an official Alliance format or endorsement.

## Try it

The validator only needs Python 3 from the standard library:

```bash
./tools/validate-fixtures
```

It checks five valid examples, seven rejected examples, deterministic evidence
identities, duplicate JSON keys, and the lifecycle rules. The canonicalization,
security, and lifecycle documents explain the rules behind those checks.

## Repository map

```text
schemas/       JSON Schemas
fixtures/      positive, negative, replay, and lifecycle examples
docs/          canonicalization, lifecycle, security, and extensions
mappings/      proposed mapping to the Verifier Alliance model
tools/         small deterministic validator
```

The optional [Sierra-to-CASM extension](docs/extensions/sierra-to-casm.md)
describes how a proof system can add compiled-class evidence without changing
the core source-match decision. It is deliberately separate while that
interface is being worked out.

## What this repository does not do

This draft does not provide a source-upload service, import another verifier's
claim, publish source code, grant redistribution rights, or make a contract
verified. Those are implementation and governance decisions for each verifier.

## Contributing

Start with an issue before changing a normative field or changing what a
verification claim means. A schema change should include a valid fixture, a
rejected fixture for each new rule, compatibility notes, and updated docs.

Please do not submit private source, credentials, authenticated provider URLs,
internal hostnames, customer data, or raw build logs. Contributions are
licensed under Apache-2.0; see [CONTRIBUTING.md](CONTRIBUTING.md).

## Status

`v0.1-draft` is early and open for review. The goal is to make the evidence
format useful before trying to standardize it.
