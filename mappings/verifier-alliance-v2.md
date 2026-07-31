# Proposed Verifier Alliance Mapping

**Proposed mapping; not reviewed or endorsed by the Verifier Alliance.**

The current Alliance source of truth is
[`database.sql`](https://github.com/verifier-alliance/database-specs/blob/master/database.sql).
The Alliance's V2 work is still a
[draft issue](https://github.com/verifier-alliance/database-specs/issues/30).
This document maps concepts to the current schema and notes the V2 proposals
that matter for Cairo evidence. It does not claim wire, schema, or membership
compatibility.

## Shared Model

Both formats separate source/compilation facts from chain evidence and the
verification relationship:

| Cairo evidence V0 | Alliance concept | Mapping |
| --- | --- | --- |
| `source.treeDigest` and source-file digests in a referenced material manifest | `sources.source_hash` and `compiled_contracts_sources.path` | Content-addressed source identity; V0 keeps source content outside the portable receipt and records publication rights separately. |
| `build.toolchain`, selectors, and policy digest | `compiled_contracts.compiler`, `version`, `language`, `name`, `fully_qualified_name`, `compiler_settings`, `additional_input` | One pinned compilation identity. Cairo package, target, profile, artifact, module, Scarb lock, and material closure do not have direct current Alliance columns. |
| `build.producedClassHash` | `compiled_contracts` creation/runtime code hashes | Compiled artifact identity, but a Starknet Sierra class hash is not EVM creation or runtime bytecode. |
| `subject.declarationEvidence` | `contract_deployments` | Authoritative chain occurrence. A Starknet class declaration is not a contract deployment and may precede or outlive all address uses. |
| `outcome.status=exact_match` | `verified_contracts` match relationship | Links qualifying compilation evidence to chain evidence. V0 requires canonical class-hash equality and has no EVM transformations. |
| `verifier.name` and policy | `created_by`; proposed V2 verifier-aware uniqueness | Identifies the contributing verifier. Authentication and a verifier registry remain future work in V0. |
| `recordId` / `evidenceDigest` | database uniqueness and content hashes | Deterministic replay identity. V0 publishes an explicit portable digest rather than relying on database-generated IDs. |
| lifecycle event | append-only policy; V2 "better verification" discussion | V0 defines explicit revocation and supersession records. The current Alliance schema updates rows and has no equivalent portable lifecycle event. |

## Non-Equivalent Concepts

### Class, Not Address

Alliance `verified_contracts` maps a compilation to a deployment. Cairo V0 maps
a compilation to a finalized class declaration. Contract addresses are
optional references and never receipt identity. A single exact class receipt
may serve every address that independently resolves to that class.

### Sierra, Not EVM Bytecode

The V0 output is a canonical Starknet Sierra class hash. It does not map to
`creation_code_hash` or `runtime_code_hash`, and it does not prove the CASM
compiled-class hash Starknet executes. Sierra-to-CASM proof evidence may be a
future extension.

### No EVM Transformations

Alliance creation/runtime transformations model constructor arguments,
immutables, linked libraries, metadata, and call protection needed to reconcile
compiled and on-chain EVM bytecode. They are not used to turn a Cairo mismatch
into an exact match. V0 requires the produced canonical class hash to equal the
selected class hash.

### Finalized Declaration

V0 binds Starknet chain ID, L1-accepted declaration block, declaration
transaction, evidence source type, and response digest. Alliance deployment
evidence binds EVM chain ID, address, transaction, block, transaction index,
and deployer. These records have different semantics and should not share a
table without an explicit tagged subject type.

### Source Rights

Alliance `sources.content` stores source text. Cairo V0 can represent private or
restricted verification while publishing only digests. `visibility`,
`retention`, and `redistribution` are independent. Importing a verification
claim never grants source retrieval or redistribution rights.

## Proposed Alliance V2 Projection

If Alliance maintainers want Cairo evidence in a future schema, the least
ambiguous projection would:

1. Add a tagged chain-subject type for Starknet class declaration rather than
   encoding a class hash as an EVM address or bytecode hash.
2. Add language-specific compilation input for package, target, profile,
   artifact, module, Scarb manifest/lock, material manifest, and environment.
3. Keep content-addressed source rows and allow digest-only private source
   records without requiring public content.
4. Include contributing verifier and verifier-policy identity in verification
   uniqueness, consistent with the V2 duplicate-verifier proposal.
5. Store explicit portable evidence identity and append-only revocation or
   supersession events.
6. Reserve a typed extension relationship for optional Sierra-to-CASM proof.

Until such a design is reviewed, an exporter SHOULD produce Cairo V0 records
beside Alliance data rather than insert lossy synthetic EVM rows.

