# Reference: Error Codes

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

NPS uses a two-level error system. This page documents the **protocol error codes** — the fine-grained layer. Each code names exactly what went wrong in a specific protocol domain. The coarser layer, NPS status codes, classifies the error for transport routing; see [Reference: Status Codes](Reference-Status-Codes).

## Code Format

Every error code follows the pattern:

```
DOMAIN-CATEGORY-DETAIL
```

All uppercase, hyphen-separated. The domain prefix matches the owning protocol: `NCP`, `NWP`, `NIP`, `NDP`, or `NOP`. The category groups related failures (e.g. `AUTH`, `QUERY`, `TASK`). The detail distinguishes the specific condition.

Examples: `NCP-ANCHOR-NOT-FOUND`, `NWP-AUTH-NID-EXPIRED`, `NOP-DELEGATE-CHAIN-TOO-DEEP`.

## The Two-Level System

```
Protocol error code   →   NPS status code         →   HTTP status (overlay mode)
NCP-ANCHOR-NOT-FOUND  →   NPS-CLIENT-NOT-FOUND    →   404
NWP-BUDGET-EXCEEDED   →   NPS-LIMIT-BUDGET        →   429
NIP-CERT-EXPIRED      →   NPS-AUTH-UNAUTHENTICATED →  401
```

Clients should branch on the **NPS status code** for generic retry/backoff logic. Use the **protocol error code** for precise diagnostic messages and agent-side recovery logic (for example, detecting `NCP-ANCHOR-STALE` to trigger a schema refresh).

---

> **Added in alpha.5**
>
> The following codes were introduced in v1.0.0-alpha.5:
>
> - `NWP-RESERVED-TYPE-UNSUPPORTED` — unrecognized reserved query/subscribe `type` field value
> - `NDP-ANNOUNCE-ROLE-REMOVED` — legacy `"gateway"` value in `node_roles` (NPS-CR-0001)
> - `NDP-ANNOUNCE-ROLE-UNKNOWN` — unrecognized value in `node_roles`
> - `NWP-MANIFEST-NODE-TYPE-REMOVED` — legacy `"gateway"` value in NWM `node_type` (NPS-CR-0001)
> - `NWP-MANIFEST-NODE-TYPE-UNKNOWN` — unrecognized NWM `node_type`
> - `NIP-CERT-EXPIRED`, `NIP-CERT-REVOKED`, `NIP-CERT-SIGNATURE-INVALID`, `NIP-CERT-UNTRUSTED-ISSUER`, `NIP-CERT-CAPABILITY-MISSING`, `NIP-CERT-SCOPE-VIOLATION`, `NIP-CERT-FORMAT-INVALID`, `NIP-CERT-EKU-MISSING`, `NIP-CERT-SUBJECT-NID-MISMATCH`, `NIP-ACME-CHALLENGE-FAILED` — NIP-namespaced certificate error codes (previously handled via NWP-AUTH-* codes)
> - `NIP-REPUTATION-GOSSIP-FORK` — Merkle tree fork detected during STH gossip (NPS-RFC-0004 §4.5)
> - `NIP-REPUTATION-GOSSIP-SIG-INVALID` — peer STH signature invalid during gossip exchange
> - `NWP-TOPOLOGY-UNAUTHORIZED`, `NWP-TOPOLOGY-UNSUPPORTED-SCOPE`, `NWP-TOPOLOGY-DEPTH-UNSUPPORTED`, `NWP-TOPOLOGY-FILTER-UNSUPPORTED` — topology query errors (NPS-CR-0002)

---

## NCP Error Codes

Neural Communication Protocol — wire format and framing layer.

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NCP-ANCHOR-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | Schema referenced by `anchor_ref` does not exist; agent must re-fetch the AnchorFrame |
| `NCP-ANCHOR-SCHEMA-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | Schema in AnchorFrame is malformed |
| `NCP-ANCHOR-ID-MISMATCH` | `NPS-CLIENT-CONFLICT` | 409 | Different schemas received for the same `anchor_id` (anchor-pollution defence) |
| `NCP-ANCHOR-STALE` | `NPS-CLIENT-CONFLICT` | 409 | `anchor_ref` exists but the schema has been updated; the response carries the latest AnchorFrame via `CapsFrame.inline_anchor` |
| `NCP-FRAME-UNKNOWN-TYPE` | `NPS-CLIENT-BAD-FRAME` | 400 | Unknown frame type code |
| `NCP-FRAME-PAYLOAD-TOO-LARGE` | `NPS-LIMIT-PAYLOAD` | 413 | Payload exceeds the negotiated `max_frame_payload` |
| `NCP-FRAME-FLAGS-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | Reserved bits in the flags field are non-zero |
| `NCP-STREAM-SEQ-GAP` | `NPS-STREAM-SEQ-GAP` | 422 | StreamFrame sequence numbers are not contiguous |
| `NCP-STREAM-NOT-FOUND` | `NPS-STREAM-NOT-FOUND` | 404 | Stream referenced by `stream_id` does not exist |
| `NCP-STREAM-LIMIT-EXCEEDED` | `NPS-STREAM-LIMIT` | 429 | Maximum concurrent streams per connection exceeded |
| `NCP-STREAM-WINDOW-OVERFLOW` | `NPS-STREAM-LIMIT` | 429 | Sender continues emitting StreamFrames after the flow-control window is exhausted |
| `NCP-ENCODING-UNSUPPORTED` | `NPS-SERVER-ENCODING-UNSUPPORTED` | 415 | Requested encoding tier is not supported by this node |
| `NCP-DIFF-FORMAT-UNSUPPORTED` | `NPS-CLIENT-BAD-FRAME` | 400 | DiffFrame uses a `patch_format` the receiver does not support (e.g. `binary_bitset`) |
| `NCP-VERSION-INCOMPATIBLE` | `NPS-PROTO-VERSION-INCOMPATIBLE` | 426 | Client `min_version` is higher than the server's maximum supported version (handshake failure) |
| `NCP-ENC-NOT-NEGOTIATED` | `NPS-CLIENT-BAD-FRAME` | 400 | Received an ENC=1 frame but no E2E encryption algorithm was negotiated for the session |
| `NCP-ENC-AUTH-FAILED` | `NPS-CLIENT-BAD-FRAME` | 400 | E2E encryption auth-tag verification failed; the frame may have been tampered with |
| `NCP-PREAMBLE-INVALID` | `NPS-PROTO-PREAMBLE-INVALID` | 400 (not emitted) | Native-mode connection opened with bytes other than the constant preamble `b"NPS/1.0\n"`; server closes silently without emitting an ErrorFrame (NPS-RFC-0001) |

---

## NWP Error Codes

Neural Web Protocol — query, action, subscription, manifest, and topology layer.

### Authentication

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-AUTH-NID-SCOPE-VIOLATION` | `NPS-AUTH-FORBIDDEN` | 403 | Agent scope does not cover the target node path |
| `NWP-AUTH-NID-EXPIRED` | `NPS-AUTH-UNAUTHENTICATED` | 401 | NID certificate has expired |
| `NWP-AUTH-NID-REVOKED` | `NPS-AUTH-UNAUTHENTICATED` | 401 | NID has been revoked |
| `NWP-AUTH-NID-UNTRUSTED-ISSUER` | `NPS-AUTH-UNAUTHENTICATED` | 401 | NID issuer is not in `trusted_issuers` |
| `NWP-AUTH-NID-CAPABILITY-MISSING` | `NPS-AUTH-FORBIDDEN` | 403 | Agent is missing a capability required by the node (e.g. `nwp:query`) |
| `NWP-AUTH-ASSURANCE-TOO-LOW` | `NPS-AUTH-FORBIDDEN` | 403 | Agent's assurance level is below the node's `min_assurance_level`; response SHOULD include a `hint` pointing to a CA enrolment URL (NPS-RFC-0003) |
| `NWP-AUTH-REPUTATION-BLOCKED` | `NPS-AUTH-FORBIDDEN` | 403 | Reputation policy matched a `reject_on` rule against the requesting `subject_nid`; response SHOULD include matching `incident` + `severity` + log entry `seq` for traceability (NPS-RFC-0004) |

### Query

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-QUERY-FILTER-INVALID` | `NPS-CLIENT-BAD-PARAM` | 400 | Filter syntax is invalid or nests deeper than 8 levels |
| `NWP-QUERY-FIELD-UNKNOWN` | `NPS-CLIENT-BAD-PARAM` | 400 | `fields` references an unknown field |
| `NWP-QUERY-CURSOR-INVALID` | `NPS-CLIENT-BAD-PARAM` | 400 | Cursor cannot be decoded or has expired |
| `NWP-QUERY-REGEX-UNSAFE` | `NPS-CLIENT-BAD-PARAM` | 400 | `$regex` pattern rejected due to ReDoS risk, excessive length, or nested quantifiers |
| `NWP-QUERY-VECTOR-UNSUPPORTED` | `NPS-SERVER-UNSUPPORTED` | 501 | Node does not support vector search |
| `NWP-QUERY-AGGREGATE-UNSUPPORTED` | `NPS-SERVER-UNSUPPORTED` | 501 | Node does not support aggregate queries (`capabilities.aggregate=false`) |
| `NWP-QUERY-AGGREGATE-INVALID` | `NPS-CLIENT-BAD-PARAM` | 400 | Aggregate structure is invalid (unknown `func`, duplicate alias, missing required fields, etc.) |
| `NWP-QUERY-STREAM-UNSUPPORTED` | `NPS-SERVER-UNSUPPORTED` | 501 | Node does not support streaming queries (`capabilities.stream_query=false`) |

### Actions and Tasks

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-ACTION-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | `action_id` is not registered on the node |
| `NWP-ACTION-PARAMS-INVALID` | `NPS-CLIENT-UNPROCESSABLE` | 422 | Action params fail schema validation |
| `NWP-ACTION-IDEMPOTENCY-CONFLICT` | `NPS-CLIENT-CONFLICT` | 409 | A request with the same `idempotency_key` is already in progress |
| `NWP-TASK-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | Asynchronous task referenced by `task_id` does not exist |
| `NWP-TASK-ALREADY-CANCELLED` | `NPS-CLIENT-CONFLICT` | 409 | Task has been cancelled; further operations are not permitted |
| `NWP-TASK-ALREADY-COMPLETED` | `NPS-CLIENT-CONFLICT` | 409 | Task has completed and cannot be cancelled |
| `NWP-TASK-ALREADY-FAILED` | `NPS-CLIENT-CONFLICT` | 409 | Task has failed and cannot be cancelled |

### Subscriptions

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-SUBSCRIBE-STREAM-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | `stream_id` referenced by `unsubscribe` does not exist |
| `NWP-SUBSCRIBE-LIMIT-EXCEEDED` | `NPS-LIMIT-EXCEEDED` | 429 | Exceeded the node's maximum concurrent subscriptions |
| `NWP-SUBSCRIBE-FILTER-UNSUPPORTED` | `NPS-SERVER-UNSUPPORTED` | 501 | Node does not support subscriptions with a filter |
| `NWP-SUBSCRIBE-INTERRUPTED` | `NPS-SERVER-UNAVAILABLE` | 503 | Subscription stream terminated because the underlying data source was interrupted |
| `NWP-SUBSCRIBE-SEQ-TOO-OLD` | `NPS-CLIENT-CONFLICT` | 409 | `resume_from_seq` is outside the node's buffer window (recommended: 10 min or 10,000 records); agent must re-query from scratch before re-subscribing |

### Budget, Limits, and Graph

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-BUDGET-EXCEEDED` | `NPS-LIMIT-BUDGET` | 429 | Response would exceed the `X-NWP-Budget` limit |
| `NWP-DEPTH-EXCEEDED` | `NPS-CLIENT-BAD-PARAM` | 400 | `X-NWP-Depth` exceeds the node's permitted `max_depth` |
| `NWP-GRAPH-CYCLE` | `NPS-CLIENT-UNPROCESSABLE` | 422 | Node graph contains a cyclic reference |
| `NWP-RATE-LIMIT-EXCEEDED` | `NPS-LIMIT-RATE` | 429 | Rate limit exceeded; reset timestamp is in the `X-NWP-Rate-Reset` header |
| `NWP-NODE-UNAVAILABLE` | `NPS-SERVER-UNAVAILABLE` | 503 | Underlying data source is temporarily unavailable |

### Manifest and Node Type

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-MANIFEST-VERSION-UNSUPPORTED` | `NPS-CLIENT-BAD-PARAM` | 400 | Agent's NPS version is lower than the node's `min_agent_version` |
| `NWP-MANIFEST-NODE-TYPE-REMOVED` | `NPS-CLIENT-BAD-FRAME` | 400 | NWM `node_type` contains the removed legacy value `"gateway"` (NPS-CR-0001); use `"anchor"` or `"bridge"`. Response SHOULD include a `hint` pointing to NPS-CR-0001. |
| `NWP-MANIFEST-NODE-TYPE-UNKNOWN` | `NPS-CLIENT-BAD-FRAME` | 400 | NWM `node_type` contains an unrecognized value that is not a known-removed legacy value |

### Reserved Type and Topology (NPS-CR-0002)

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NWP-RESERVED-TYPE-UNSUPPORTED` | `NPS-SERVER-UNSUPPORTED` | 501 | `QueryFrame` or `SubscribeFrame` `type` field contains an unrecognized reserved-type identifier; this node does not implement the requested reserved operation (NWP §12) |
| `NWP-TOPOLOGY-UNAUTHORIZED` | `NPS-AUTH-FORBIDDEN` | 403 | Caller lacks permission to read this Anchor's topology (NPS-CR-0002 §12.4) |
| `NWP-TOPOLOGY-UNSUPPORTED-SCOPE` | `NPS-CLIENT-BAD-PARAM` | 400 | `topology.scope` value is not implemented by this Anchor Node |
| `NWP-TOPOLOGY-DEPTH-UNSUPPORTED` | `NPS-CLIENT-BAD-PARAM` | 400 | Requested `topology.depth` exceeds this Anchor Node's configured maximum |
| `NWP-TOPOLOGY-FILTER-UNSUPPORTED` | `NPS-CLIENT-BAD-PARAM` | 400 | `topology.filter` contains an unrecognized key or unsupported operator |

---

## NIP Error Codes

Neural Identity Protocol — certificates, trust chains, assurance levels, and reputation.

### Certificate Validation

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NIP-CERT-EXPIRED` | `NPS-AUTH-UNAUTHENTICATED` | 401 | Certificate has expired (`expires_at < now`) |
| `NIP-CERT-REVOKED` | `NPS-AUTH-UNAUTHENTICATED` | 401 | Certificate has been revoked (found in CRL or OCSP) |
| `NIP-CERT-SIGNATURE-INVALID` | `NPS-AUTH-UNAUTHENTICATED` | 401 | Certificate signature verification failed |
| `NIP-CERT-UNTRUSTED-ISSUER` | `NPS-AUTH-UNAUTHENTICATED` | 401 | Issuer is not in `trusted_issuers` |
| `NIP-CERT-CAPABILITY-MISSING` | `NPS-AUTH-FORBIDDEN` | 403 | Certificate is missing a capability required by the node |
| `NIP-CERT-SCOPE-VIOLATION` | `NPS-AUTH-FORBIDDEN` | 403 | Certificate scope does not cover the target path or operation |
| `NIP-CERT-FORMAT-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | `IdentFrame.cert_chain` is not DER-encoded X.509 or fails ASN.1 parsing (NPS-RFC-0002 §4.3) |
| `NIP-CERT-EKU-MISSING` | `NPS-CLIENT-BAD-FRAME` | 400 | Required NPS EKU (`agent-identity` or `node-identity`) is absent or non-critical on the leaf cert (NPS-RFC-0002 §4.1/§4.3) |
| `NIP-CERT-SUBJECT-NID-MISMATCH` | `NPS-CLIENT-BAD-FRAME` | 400 | X.509 leaf cert subject CN / SAN URI does not match `IdentFrame.nid` (NPS-RFC-0002 §4.3) |

### CA Operations

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NIP-CA-NID-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | NID does not exist in the CA database |
| `NIP-CA-NID-ALREADY-EXISTS` | `NPS-CLIENT-CONFLICT` | 409 | NID already exists (duplicate registration) |
| `NIP-CA-SERIAL-DUPLICATE` | `NPS-CLIENT-CONFLICT` | 409 | Certificate serial number already in use |
| `NIP-CA-RENEWAL-TOO-EARLY` | `NPS-CLIENT-BAD-PARAM` | 400 | More than 7 days until expiry; renewal window not yet open |
| `NIP-CA-SCOPE-EXPANSION-DENIED` | `NPS-AUTH-FORBIDDEN` | 403 | Requested scope exceeds the parent scope (delegation-chain violation) |
| `NIP-ACME-CHALLENGE-FAILED` | `NPS-CLIENT-BAD-FRAME` | 400 | ACME `agent-01` challenge validation failed (token mismatch, signature failure, or replay — NPS-RFC-0002 §4.4) |

### OCSP and Trust

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NIP-OCSP-UNAVAILABLE` | `NPS-SERVER-UNAVAILABLE` | 503 | OCSP service temporarily unavailable |
| `NIP-TRUST-FRAME-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | TrustFrame signature or format is invalid |

### Assurance and Reputation

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NIP-ASSURANCE-MISMATCH` | `NPS-CLIENT-BAD-FRAME` | 400 | `IdentFrame.assurance_level` does not match the cert extension `id-nid-assurance-level` (downgrade-attack defence — NPS-RFC-0003, NPS-3 §5.1.1) |
| `NIP-ASSURANCE-UNKNOWN` | `NPS-CLIENT-BAD-FRAME` | 400 | `assurance_level` is outside the defined enum (`anonymous` / `attested` / `verified`) |
| `NIP-REPUTATION-ENTRY-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | Reputation log entry signature fails verification or canonical (RFC 8785 JCS) form is malformed (NPS-RFC-0004) |
| `NIP-REPUTATION-LOG-UNREACHABLE` | `NPS-DOWNSTREAM-UNAVAILABLE` | 502/503 | A log operator referenced by the node's `reputation_policy` cannot be reached during admission evaluation (NPS-RFC-0004) |
| `NIP-REPUTATION-GOSSIP-FORK` | `NPS-SERVER-INTERNAL` | 500 | Cross-peer STH consistency check failed; possible Merkle tree fork detected (NPS-RFC-0004 §4.5) |
| `NIP-REPUTATION-GOSSIP-SIG-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | Peer STH signature verification failed during gossip exchange (NPS-RFC-0004 §4.5) |

---

## NDP Error Codes

Neural Discovery Protocol — address resolution, announcement, and graph synchronization.

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NDP-RESOLVE-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | `nwp://` address cannot be resolved to a physical endpoint |
| `NDP-RESOLVE-AMBIGUOUS` | `NPS-CLIENT-CONFLICT` | 409 | Resolution result is conflicting (multiple inconsistent registrations) |
| `NDP-RESOLVE-TIMEOUT` | `NPS-SERVER-TIMEOUT` | 408/504 | Resolution request timed out |
| `NDP-ANNOUNCE-SIGNATURE-INVALID` | `NPS-AUTH-UNAUTHENTICATED` | 401 | AnnounceFrame signature verification failed |
| `NDP-ANNOUNCE-NID-MISMATCH` | `NPS-CLIENT-BAD-FRAME` | 400 | NID in AnnounceFrame does not match the signing certificate |
| `NDP-ANNOUNCE-ROLE-REMOVED` | `NPS-CLIENT-BAD-FRAME` | 400 | AnnounceFrame `node_roles` contains the removed legacy value `"gateway"` (NPS-CR-0001); use `"anchor"` or `"bridge"`. Response SHOULD include a `hint` pointing to NPS-CR-0001. |
| `NDP-ANNOUNCE-ROLE-UNKNOWN` | `NPS-CLIENT-BAD-FRAME` | 400 | AnnounceFrame `node_roles` contains an unrecognized value that is not a known-removed legacy value |
| `NDP-GRAPH-SEQ-GAP` | `NPS-STREAM-SEQ-GAP` | 422 | GraphFrame sequence numbers are not contiguous |
| `NDP-REGISTRY-UNAVAILABLE` | `NPS-SERVER-UNAVAILABLE` | 503 | NDP Registry temporarily unavailable |

---

## NOP Error Codes

Neural Orchestration Protocol — DAG task dispatch, delegation, synchronization, and streaming.

### Task Lifecycle

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NOP-TASK-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | 404 | `task_id` does not exist |
| `NOP-TASK-TIMEOUT` | `NPS-SERVER-TIMEOUT` | 408/504 | Overall task execution timed out |
| `NOP-TASK-DAG-INVALID` | `NPS-CLIENT-BAD-FRAME` | 400 | DAG structure is invalid (missing entry/exit node, field error, etc.) |
| `NOP-TASK-DAG-CYCLE` | `NPS-CLIENT-BAD-FRAME` | 400 | DAG contains a cycle |
| `NOP-TASK-DAG-TOO-LARGE` | `NPS-CLIENT-BAD-FRAME` | 400 | DAG node count exceeds the limit (default: 32) |
| `NOP-TASK-ALREADY-COMPLETED` | `NPS-CLIENT-CONFLICT` | 409 | Task has completed; cannot be resubmitted |
| `NOP-TASK-CANCELLED` | `NPS-CLIENT-CONFLICT` | 409 | Task has been cancelled |

### Delegation

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NOP-DELEGATE-SCOPE-VIOLATION` | `NPS-AUTH-FORBIDDEN` | 403 | `delegated_scope` exceeds the parent agent's scope |
| `NOP-DELEGATE-REJECTED` | `NPS-CLIENT-UNPROCESSABLE` | 422 | Target agent declined the delegation (insufficient capability or overload); response includes `retry_after_ms` |
| `NOP-DELEGATE-CHAIN-TOO-DEEP` | `NPS-CLIENT-BAD-PARAM` | 400 | Delegation chain depth exceeds the limit (default: 3 levels) |
| `NOP-DELEGATE-TIMEOUT` | `NPS-SERVER-TIMEOUT` | 408/504 | Subtask did not complete before `deadline_at` |

### Synchronization and Streaming

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NOP-SYNC-TIMEOUT` | `NPS-SERVER-TIMEOUT` | 408/504 | SyncFrame timed out waiting for dependent tasks |
| `NOP-SYNC-DEPENDENCY-FAILED` | `NPS-CLIENT-UNPROCESSABLE` | 422 | Dependency subtask has failed and failure count exceeds the K-of-N tolerance |
| `NOP-STREAM-SEQ-GAP` | `NPS-STREAM-SEQ-GAP` | 422 | AlignStream sequence numbers are not contiguous |
| `NOP-STREAM-NID-MISMATCH` | `NPS-AUTH-UNAUTHENTICATED` | 401 | AlignStream `sender_nid` does not match the connection identity |

### Resources and Conditions

| Code | NPS Status Code | HTTP Equivalent | Description |
|------|-----------------|-----------------|-------------|
| `NOP-RESOURCE-INSUFFICIENT` | `NPS-SERVER-UNAVAILABLE` | 503 | Preflight found one or more Worker Agents lack sufficient resources (CGN or capabilities) |
| `NOP-CONDITION-EVAL-ERROR` | `NPS-CLIENT-BAD-PARAM` | 400 | DAG node `condition` expression failed to evaluate (syntax error or missing referenced field) |
| `NOP-INPUT-MAPPING-ERROR` | `NPS-CLIENT-UNPROCESSABLE` | 422 | `input_mapping` JSONPath could not be resolved or target field is missing |

---

## Disambiguation: NWP-ACTION-NOT-FOUND vs NWP-RESERVED-TYPE-UNSUPPORTED

These two codes cover different failure modes within `QueryFrame`, `ActionFrame`, and `SubscribeFrame`, and are easy to confuse.

**`NWP-ACTION-NOT-FOUND`** applies when:

- The caller names a specific action via `action_id`
- The `action_id` is a known operand type (looked up in the node's action registry)
- But no action with that identifier is registered at this endpoint

Use this code when the error is "I know what an action is, but this particular one does not exist here." Maps to `NPS-CLIENT-NOT-FOUND` (HTTP 404).

**`NWP-RESERVED-TYPE-UNSUPPORTED`** applies when:

- The `type` field of a `QueryFrame` or `SubscribeFrame` carries a reserved-namespace identifier (e.g. `topology.snapshot`, `topology.stream`)
- The node does not implement that entire reserved operation class

Use this code when the error is "I don't know how to handle this type of operation at all." The `type` field is the unknown operand, not `action_id`. Maps to `NPS-SERVER-UNSUPPORTED` (HTTP 501) because it represents a server capability gap, not a missing client-side resource.

**Quick rule**: "I don't have this action by name" → `NWP-ACTION-NOT-FOUND`. "I don't implement this class of operation" → `NWP-RESERVED-TYPE-UNSUPPORTED`.

---

## See Also

- [Reference: Status Codes](Reference-Status-Codes) — the coarse two-level classification and HTTP mapping
- [Protocol NWP](Protocol-NWP) — NWP action and query semantics in full

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
