# Canonicalization

The key words MUST, MUST NOT, REQUIRED, SHOULD, and MAY are interpreted as in
RFC 2119 and RFC 8174.

## JSON

Implementations MUST parse records as I-JSON and serialize the `evidence`
object with RFC 8785 JSON Canonicalization Scheme (JCS) before hashing it.

The V0 profile adds these restrictions:

- duplicate object keys, non-finite numbers, floating-point values, and
  integers outside `[-9007199254740991, 9007199254740991]` MUST be rejected;
- strings and object keys MUST be valid UTF-8 in Unicode NFC form and MUST NOT
  contain unpaired surrogates or C0 control characters other than characters
  escaped by JSON;
- Starknet felts MUST use `0x` followed by exactly 64 lowercase hexadecimal
  digits and MUST be smaller than the Starknet field prime;
- digests MUST use `sha256:` followed by exactly 64 lowercase hexadecimal
  digits; and
- timestamps MUST use UTC `YYYY-MM-DDTHH:MM:SSZ`.

`evidenceDigest` is:

```text
sha256:<lowercase hex SHA-256(JCS(record.evidence))>
```

`recordId` is:

```text
urn:starknet-source-verification:<evidenceDigest>
```

`recordId`, `evidenceDigest`, and `timing` do not participate in
`evidenceDigest`. A replay of identical evidence therefore has the same
identity even when the build timing differs. Timing remains evidence metadata
and MUST NOT be used to silently replace the first accepted observation.

## Source Tree

The complete source tree digest MUST be computed before an archive is admitted
to a verifier.

For every regular file:

1. Decode the relative path as valid UTF-8 and normalize it to NFC.
2. Replace platform separators with `/`.
3. Reject empty segments, `.`, `..`, absolute paths, duplicate normalized
   paths, symlinks, hard links, devices, FIFOs, sockets, and sparse files.
4. Preserve file bytes exactly. Line endings and final newlines are not
   normalized.
5. Normalize the mode to `100755` when any executable bit is set and `100644`
   otherwise. Ownership, group, timestamps, ACLs, and extended attributes are
   excluded.
6. Compute the lowercase hexadecimal SHA-256 of the file bytes.

Sort records by the UTF-8 bytes of the normalized path. Encode each record as:

```text
<path UTF-8><NUL><mode ASCII><NUL><size decimal ASCII><NUL><file SHA-256 hex><LF>
```

Concatenate the records without a header or trailer. `treeDigest` is the
`sha256:` form of the SHA-256 of those bytes. An empty source tree is invalid.

`bundleDigest` hashes the admitted archive bytes. It may differ for archives
that unpack to the same source tree; `treeDigest` is the stable source-content
identity.

`scarbManifestDigest` and `scarbLockDigest` hash the exact admitted bytes of
`Scarb.toml` and `Scarb.lock`. A missing lockfile is not equivalent to an empty
lockfile and MUST be rejected for an exact verification.

## Build Materials

`materialManifestDigest` binds a verifier-defined manifest that enumerates all
compiler binaries and dependency materials available to the offline build.
The manifest format is outside V0, but its digest is required. A verifier MUST
reject undeclared or mutable network-resolved build material before claiming
`exact_match`.

