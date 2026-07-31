# Evidence Lifecycle

Evidence and lifecycle records are append-only. Published records MUST NOT be
silently changed or deleted.

## Actions

`revoke` withdraws reliance on an evidence record. It preserves the original
record and states why consumers should no longer treat it as effective. A
revocation MUST NOT include `replacementEvidenceDigest`.

`supersede` points from an older evidence record to a stronger or corrected
record. It MUST include `replacementEvidenceDigest`, and the replacement MUST
not equal `subjectEvidenceDigest`.

A lifecycle event is identified by the RFC 8785 JCS SHA-256 digest of its
`event` object:

```text
eventDigest = sha256:<SHA-256(JCS(record.event))>
eventId = urn:starknet-source-verification-event:<eventDigest>
```

The effective timestamp participates in event identity. Events with different
effective timestamps are distinct even if they otherwise express the same
transition.

## Authorization

V0 records issuer identity and policy but does not define signatures or a
global authority registry. Consumers MUST authenticate lifecycle publishers
out of band and MUST NOT accept an event merely because it is structurally
valid.

An implementation SHOULD accept lifecycle events only from the verifier that
published the subject evidence or from a documented governance authority. A
future namespaced extension may bind a signature or transparency-log entry.

## Effective Status

For one `evidenceDigest`, a consumer:

1. Validates the original evidence record.
2. Selects only authenticated, structurally valid lifecycle events that bind
   that digest.
3. Orders events by `effectiveAt`, then by `eventDigest` as a deterministic tie
   breaker.
4. Treats any effective `revoke` as withdrawn.
5. Treats `supersede` as a pointer, validates the replacement independently,
   and follows the chain.
6. Rejects cycles, missing replacements, replacement self-references, and
   chains longer than 32 events.

Revocation is not undone by publishing a later exact-match record. A new record
has its own identity and history.

## Replay

Evidence identity excludes run timing. Identical exact evidence rebuilt later
has the same `evidenceDigest` and `recordId`. Implementations SHOULD preserve
the first accepted observation and MAY attach later replay observations
outside the portable evidence record.

