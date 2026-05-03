# Protocol: NIP — Neural Identity Protocol

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

**Spec**: `spec/NPS-3-NIP.md` v0.6 · **Port**: 17433 (shared) / 17435 (optional dedicated)

NIP is the identity and trust backbone of NPS. It issues verifiable Neural Identities (NIDs) to every AI Agent, NWP Node, and human Operator, carries capability declarations and scope-bound permissions, supports trust-chain propagation across organizations, and provides real-time revocation. The analogy is TLS/PKI: where TLS secures the transport channel, NIP secures the *identity of the communicating parties* regardless of transport.

Related: [SDK Identity and Authentication](SDK-Identity-and-Authentication) | [Operator Reputation Log](Operator-Reputation-Log) | [Protocol NWP](Protocol-NWP)

---

## NID Format

A Neural Identity (NID) is a URN with the following structure:

```
urn:nps:{entity-type}:{issuer-domain}:{identifier}

Examples:
  urn:nps:agent:ca.innolotus.com:550e8400-e29b-41d4    <- AI agent
  urn:nps:node:api.myapp.com:products                   <- NWP node
  urn:nps:org:mycompany.com                             <- Organization CA
```

`entity-type` is one of `agent`, `node`, or `org`. `issuer-domain` is an RFC 1034 domain. `identifier` may contain alphanumerics, hyphens, underscores, and dots.

### NID Lifecycle

1. **Issue:** The Org CA issues an `IdentFrame` containing the NID, public key, capabilities, scope, and CA signature. Agent certificates have 30-day validity (auto-renewal supported); Node certificates have 90-day validity; Operator certificates have 1-year validity.
2. **Use:** The Agent presents its `IdentFrame` on every new connection. The receiving Node verifies the signature, checks expiry, validates capabilities, and confirms scope coverage.
3. **Revoke:** The CA issues a `RevokeFrame` (0x22). Revocation takes effect immediately. Nodes verify revocation via OCSP (when configured) or local CRL.

### CA Hierarchy

```
Root CA (offline, rarely used)
    |
    +-- Org Intermediate CA (one per organization, 1-year validity)
            +-- Agent Certificate     (30-day validity, auto-renewal)
            +-- Node Certificate      (90-day validity)
            +-- Operator Certificate  (1-year validity, bound to MFA)
```

Primary signature algorithm: **Ed25519** (32-byte keys, high-frequency verification performance). Fallback: ECDSA P-256 for compatibility scenarios. Public keys are encoded as `{algorithm}:{base64url(DER)}`, e.g. `ed25519:MCowBQYDK2VwAyEA...`.

---

## Frame Types

### IdentFrame (0x20)

The Agent identity declaration frame. Sent as the handshake frame on every new native-mode connection, and included in HTTP-mode requests via headers.

**Key fields:**

| Field | Type | Description |
|-------|------|-------------|
| `nid` | string | Agent NID |
| `pub_key` | string | Public key in `{alg}:{base64url}` format |
| `capabilities` | array | Capabilities held by this agent (see Capability Registry below) |
| `scope` | object | Access scope: `{nodes: [...], actions: [...], max_token_budget: N}` |
| `issued_by` | string | Issuer NID (Org CA) |
| `issued_at` / `expires_at` | string | ISO 8601 UTC timestamps |
| `serial` | string | Certificate serial (globally unique per Org CA, hex) |
| `signature` | string | CA's Ed25519 signature over the canonical frame (with `signature` field excluded), computed per RFC 8785 JCS |
| `assurance_level` | string | `"anonymous"` / `"attested"` / `"verified"` — see Three-Tier Assurance Levels below |
| `metadata` | object | Optional: `{model_family, tokenizer, runtime}` — not included in signature computation; used for CGN auto-matching |

The signature is computed over the canonical JSON of the `IdentFrame` with the `signature` field removed and object keys sorted alphabetically with no extra whitespace (RFC 8785 JCS).

### TrustFrame (0x21)

Cross-CA trust-chain propagation. Allows one organization (grantor) to extend trust to another organization's CA (grantee) for a defined scope and node set. Carries `grantor_nid`, `grantee_ca`, `trust_scope`, `nodes`, `expires_at`, and a signature from the grantor. This is a commercial feature used in NPS Cloud multi-org deployments.

### RevokeFrame (0x22)

Revokes an NID or a specific capability. Carries `target_nid`, `serial`, `reason`, `revoked_at`, and a CA signature. Takes effect immediately upon receipt by a Node. Valid `reason` values: `key_compromise`, `ca_compromise`, `affiliation_changed`, `superseded`, `cessation_of_operation`.

---

## Capability Registry

Capabilities are declared in `IdentFrame.capabilities` as a string array. Nodes check this array before granting access to specific operations.

| Capability | Description |
|------------|-------------|
| `nwp:query` | May query Memory Nodes |
| `nwp:action` | May invoke Action Nodes |
| `nwp:stream` | May receive `StreamFrame` responses |
| `ncp:stream` | May initiate NCP streaming |
| `nop:delegate` | May delegate subtasks to other Agents |
| `nop:orchestrate` | May act as an Orchestrator and emit `TaskFrame`s |
| `topology:read` | May read Anchor Node topology data via `topology.snapshot` / `topology.stream` (NPS-2 §12). Added in NIP v0.6 (alpha.5). Anchor Nodes MUST require this capability at Phase 1–2 per NPS-2 §12.4. Self-declared and key-signed; CA-attested role binding deferred to Phase 3. |

Capability declarations are signed by the issuing CA inside `IdentFrame.signature`. An Agent cannot add capabilities to its own certificate — only the CA can grant them at issuance time.

---

## Three-Tier Assurance Levels (NPS-RFC-0003)

NPS-RFC-0003 (Accepted, Phase 1 landed) defines three assurance levels for Agent identities, modeled on NIST SP 800-63 IAL and CA/B Forum DV/OV/EV certificates. The level travels in `IdentFrame.assurance_level` and is the source of truth for Node policy decisions via `NWM.min_assurance_level`.

| Level | Enum value | Minimum CA criteria | Typical use |
|-------|------------|---------------------|-------------|
| L0 | `"anonymous"` | Self-signed NID, or CA-signed without out-of-band identity binding | Hobbyist Agents, dev/test, free read-only endpoints |
| L1 | `"attested"` | NID signed by an RFC-0002-compliant CA; CA attests possession of the NID private key (ACME `agent-01` challenge); contact email or domain verified | Most production Agents; default rate-limit tier |
| L2 | `"verified"` | L1 criteria plus CA binds the operator's legal identity (corporate registration or signed AaaS-operator attestation) | Regulated integrations, paid premium tiers, contract-grade orchestration |

The levels form an **ordered enum**: `anonymous < attested < verified`. A request whose level is below the Node's required level MUST be rejected with `NWP-AUTH-ASSURANCE-TOO-LOW` (`NPS-AUTH-FORBIDDEN`).

Default is `"anonymous"` — pre-RFC-0003 NIDs and any NID lacking the `assurance_level` field are treated as L0.

### AssuranceLevel.fromWire("") — Empty String Fix (alpha.5)

Prior to alpha.5, some SDKs serialized a missing or null `assurance_level` as an empty string `""` on the wire instead of omitting the field. This caused parsing errors in receiving SDKs that did not handle the empty-string case.

**Spec fix (cross-SDK parity):** an `assurance_level` field containing an empty string `""` MUST be treated as equivalent to the field being absent, meaning the level resolves to `"anonymous"`. This fix was applied across all six SDKs (.NET, Python, TypeScript, Java, Rust, Go) in alpha.5.

### Forward Compatibility: NIP-ASSURANCE-UNKNOWN

An implementation receiving an `assurance_level` value not in the defined enum (`anonymous`, `attested`, `verified`) MUST return `NIP-ASSURANCE-UNKNOWN` (`NPS-CLIENT-BAD-FRAME`). Implementations MUST NOT silently demote an unknown value to `anonymous` — doing so would create a security loophole when future higher-assurance levels are introduced by a later spec revision.

---

## X.509 + ACME Path (NPS-RFC-0002)

NPS-RFC-0002 proposes replacing the NIP custom certificate format with standard X.509v3 certificates and moving issuance to ACME (RFC 8555) with an `agent-01` challenge type. The primary motivation is tooling reuse: OpenSSL, step-ca, HashiCorp Vault PKI, HSM vendors, and cert-manager for Kubernetes can then sign, validate, and store NIP certificates without custom integration.

**Current status: EXPERIMENTAL — Draft.** The prototype uses the provisional, unregistered OID `1.3.6.1.4.1.99999.1`. This OID:
- MUST NOT be used for conformance testing at any compliance level
- MUST NOT be used for production NID issuance
- MUST NOT be used in cross-organization interoperability tests

RFC-0002 will be promoted from Draft to Proposed/Accepted only after LabAcacia's IANA PEN is assigned and the provisional OID is replaced throughout all SDKs and the NIP CA Server. All certificates issued under the provisional OID will need to be revoked and reissued at that point.

---

## Reputation Log Entry Shape (NPS-RFC-0004)

NPS-RFC-0004 (Accepted, Phase 1 landed) defines a Certificate-Transparency-style append-only log for NID behavioral incidents. Where `assurance_level` answers "who is this Agent?", reputation log entries answer "how has this NID behaved?" — published so any party (Node, auditor, downstream operator) can assess a NID's track record without a pre-existing relationship with the publisher.

A reputation entry is a signed JSON object with 12 fields:

| Field | Description |
|-------|-------------|
| `v` | Schema version (`1`) |
| `log_id` | NID of the log operator that appended this entry |
| `seq` | Monotonically increasing per-`log_id` sequence number (set by log operator on append) |
| `timestamp` | Log operator's commit time (RFC 3339 UTC) |
| `subject_nid` | The NID this entry is about |
| `incident` | Incident type (see vocabulary below) |
| `severity` | `info` / `minor` / `moderate` / `major` / `critical` |
| `window` | Optional observation window `{start, end}` |
| `observation` | Optional machine-readable per-incident detail (e.g. `{requests: 45000, threshold: 300}`) |
| `evidence_ref` | Optional URL where richer evidence lives |
| `evidence_sha256` | Optional SHA-256 of the evidence blob for tamper detection |
| `issuer_nid` | NID of the party making the assertion |
| `signature` | Ed25519 signature by `issuer_nid`'s private key over the entry minus `signature`, canonicalized per RFC 8785 JCS |

The dual-signature model: the issuer signs the entry; the log operator then re-signs the full entry (including the issuer signature) with the `log_id` key as an ordering commitment.

### Incident Vocabulary

| Value | Meaning |
|-------|---------|
| `cert-revoked` | CA revoked the subject NID's certificate |
| `rate-limit-violation` | Sustained violation of published rate limits |
| `tos-violation` | Violated an AaaS Anchor Node's published terms |
| `scraping-pattern` | Behavior matched scraper heuristics |
| `payment-default` | CGN or fiat default on a committed transaction |
| `contract-dispute` | Unresolved contractual breach on an async NOP task |
| `impersonation-claim` | Third-party claim that subject NID is impersonating them |
| `positive-attestation` | Explicit positive signal (e.g. independent audit passed) |

Receivers MUST treat unknown `incident` values as opaque pass-through (forward compatibility).

---

## Phase 3 STH Gossip (alpha.5)

Phase 3 of NPS-RFC-0004 introduced Signed Tree Head (STH) gossip to enable fork detection across independent log operators. The `nps-ledger` daemon exposes:

```
GET /v1/log/gossip/sth
```

This endpoint returns the current STH that the ledger has committed (tree size, root hash, timestamp, and signature). Other log operators fetch STHs from peers and compare root hashes. If two operators disagree on the tree root at the same tree size, a fork is detected.

**Error code for fork detection:** `NIP-REPUTATION-GOSSIP-FORK` — emitted when the ledger detects a conflicting STH from a gossip peer, indicating either log corruption or a malicious log operator attempt to present different views to different observers.

The STH gossip protocol is implemented in the `nps-ledger` daemon (`GossipState` + `GossipService`) and shipped in alpha.5 with 13 test cases covering STH fetch, gossip exchange, and fork detection.

---

## NIP Verification Flow

When a Node receives an `IdentFrame`:

1. Check `expires_at > now` — expired: `NIP-CERT-EXPIRED`
2. Check `issued_by` is in the NWM `trusted_issuers` — not trusted: `NIP-CERT-UNTRUSTED-ISSUER`
3. Verify `signature` with the issuer CA's public key — invalid: `NIP-CERT-SIGNATURE-INVALID`
4. OCSP lookup (when NWM configures `ocsp_url`) or local CRL check — revoked: `NIP-CERT-REVOKED`
5. Check `capabilities` contains what the node requires — missing: `NIP-CERT-CAPABILITY-MISSING`
6. Check `scope.nodes` covers the target node path — not covered: `NWP-AUTH-NID-SCOPE-VIOLATION`

All checks pass → authorize the request.

---

## NIP CA Server OSS API

The reference NIP CA Server (`tools/nip-ca-server/`) exposes:

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/agents/register` | Register an agent; returns NID + IdentFrame |
| POST | `/v1/agents/{nid}/renew` | Renew certificate (up to 7 days before expiry) |
| POST | `/v1/agents/{nid}/revoke` | Revoke an NID |
| GET | `/v1/agents/{nid}/verify` | Verify NID validity (OCSP lookup) |
| POST | `/v1/nodes/register` | Register an NWP node |
| GET | `/v1/ca/cert` | CA public-key certificate |
| GET | `/v1/crl` | Certificate revocation list |
| GET | `/.well-known/nps-ca` | CA discovery endpoint |

---

## Error Codes

| Error Code | NPS Status | Description |
|------------|------------|-------------|
| `NIP-CERT-EXPIRED` | `NPS-AUTH-UNAUTHENTICATED` | Certificate has expired |
| `NIP-CERT-REVOKED` | `NPS-AUTH-UNAUTHENTICATED` | Certificate has been revoked |
| `NIP-CERT-SIGNATURE-INVALID` | `NPS-AUTH-UNAUTHENTICATED` | Certificate signature verification failed |
| `NIP-CERT-UNTRUSTED-ISSUER` | `NPS-AUTH-UNAUTHENTICATED` | Issuer not in the trust list |
| `NIP-CERT-CAPABILITY-MISSING` | `NPS-AUTH-FORBIDDEN` | Certificate missing a required capability |
| `NIP-CERT-SCOPE-VIOLATION` | `NPS-AUTH-FORBIDDEN` | Certificate scope does not cover the target path |
| `NIP-ASSURANCE-MISMATCH` | `NPS-CLIENT-BAD-FRAME` | `IdentFrame.assurance_level` disagrees with the cert extension `id-nid-assurance-level` (downgrade-attack defense; enforced in Phase 3) |
| `NIP-ASSURANCE-UNKNOWN` | `NPS-CLIENT-BAD-FRAME` | `assurance_level` carries a value outside the defined enum |
| `NIP-REPUTATION-ENTRY-INVALID` | `NPS-CLIENT-BAD-FRAME` | Reputation log entry signature fails verification or canonical form is malformed |
| `NIP-REPUTATION-LOG-UNREACHABLE` | `NPS-DOWNSTREAM-UNAVAILABLE` | A log operator referenced by a Node's `reputation_policy` cannot be reached during admission evaluation |
| `NIP-CA-NID-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | NID does not exist |
| `NIP-CA-NID-ALREADY-EXISTS` | `NPS-CLIENT-CONFLICT` | NID already registered (duplicate registration) |
| `NIP-CA-RENEWAL-TOO-EARLY` | `NPS-CLIENT-BAD-PARAM` | Renewal window not yet open |
| `NIP-CA-SCOPE-EXPANSION-DENIED` | `NPS-AUTH-FORBIDDEN` | Requested scope exceeds the parent scope (no-scope-expansion principle) |
| `NIP-OCSP-UNAVAILABLE` | `NPS-SERVER-UNAVAILABLE` | OCSP service temporarily unavailable |

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
