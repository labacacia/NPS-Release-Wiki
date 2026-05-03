# Operator: Node Conformance & Certification

> **Audience:** Operators + node implementers
> **Status:** ✅ Content complete — v1.0.0-alpha.5.2
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

NPS has two orthogonal compliance profiles. This page explains how they relate and how to run the conformance test suite and publish a self-attestation.

---

## Node-Profile vs AaaS-Profile: orthogonal

| Profile | Question answered | Who cares |
|---------|-------------------|-----------|
| **NPS-Node Profile** | "Is my *host* (daemon / embedded runtime) a conformant participant in the NPS network?" | Node implementers; operators running daemons |
| **NPS-AaaS Profile** | "Does my *service* expose the right NPS endpoints to AI agents?" | AaaS vendors; teams building agent-facing APIs |

A deployment may carry **both** badges, **either**, or **neither**. The two are additive, not mutually exclusive. The cross-profile contract (introduced in alpha.5) is:

- An **AaaS Anchor Node operator claiming L2-08** MUST satisfy **Node-Profile L1** for the host running the Anchor.
- An **Anchor that maintains an active member registry** (i.e., uses `nps-registry` or equivalent) SHOULD also satisfy **Node-Profile L2**.

In other words: topology query capability (L2-08) implies the underlying host has passed the baseline L1 node test suite. A registry-backed Anchor is held to the interactive L2 standard.

---

## Node-Profile Level 1: 21 test cases

The `NPS-Node-L1` conformance suite (`spec/services/conformance/NPS-Node-L1.md`) defines **21 `TC-N1-*` test cases** in five domains:

| Domain | Cases | What they cover |
|--------|-------|-----------------|
| NCP — Wire format | TC-N1-NCP-01 through TC-N1-NCP-04 | Tier-1 JSON frame round-trip; Hello+Anchor handshake; loopback listener default; Tier-2 negotiation hygiene |
| NIP — Identity | TC-N1-NIP-01 through TC-N1-NIP-04 | Root keypair generation and permissions; IdentFrame sign/verify; NID format; sub-NID issuance (optional) |
| NDP — Discovery | TC-N1-NDP-01 through TC-N1-NDP-04 | AnnounceFrame carries `activation_mode = "ephemeral"`; AnnounceFrame signature; ResolveFrame response; GraphFrame subscription (optional) |
| NWP — Inbox and delivery | TC-N1-NWP-01 through TC-N1-NWP-05 | Inbox accepts ActionFrame; inbox persists across restart; FIFO pull order; 100 QPS baseline; push path (optional) |
| Observability | TC-N1-OBS-01 through TC-N1-OBS-03 | Structured log entry per frame direction; required log fields; log destination flexibility |

Three cases (NIP-04, NDP-04, NWP-05) may be recorded as **N/A** if the implementation declines the optional capability; they do not count as failures. All other 18 cases MUST be `pass`.

### Test methodology: paired-peer

Every L1 case is run with a **peer** — any implementation that already passes L1, or the .NET reference SDK (`NPS.Core` + `NPS.NDP` + `NPS.NIP` + `NPS.NWP` at v1.0.0-alpha.3 or later). Run every case against the Implementation Under Test (IUT) paired with the peer. Use a fresh IUT state (wipe the key store, registry, and inbox) between cases unless the case is explicitly continuation-oriented.

Test environment requirements:
- Network: loopback only; no external DNS or inter-host routing required.
- Clock: wall clock within ±5 s of the peer (ISO 8601 UTC timestamps).
- File system: writable directory for the IUT's key store and inbox persistence.
- Wire encoding: Tier-1 JSON MUST be exercised; Tier-2 MsgPack cases are deferred to L2.

---

## Node-Profile Level 2: additional test cases

Level 2 extends L1 with interactive requirements. Detailed requirement IDs (`N2-NCP-*` etc.) are tracked under NPS-Roadmap Phase 2; the conformance document (`spec/services/conformance/NPS-Node-L2.md`) is planned but not yet published.

Headline L2 additions on top of L1:

| Domain | Addition |
|--------|---------|
| NCP | Tier-2 MsgPack MUST be negotiated and used |
| NIP | Trust-chain validation against a configured trust anchor MUST succeed |
| NDP | GraphFrame subscription MUST work; node SHOULD react to incremental changes within the `seq` window |
| NWP | ActionFrame push and Subscribe MUST work; Anchor Nodes MUST respond to `topology.snapshot` / `topology.stream` (NPS-2 §12) |
| NOP | TaskFrame and DelegateFrame MUST be accepted |
| Activation | `resident` mode MUST be supported |
| Observability | Prometheus-style metrics endpoint MUST be exposed (frame counters, inbox depth, connection count, handshake latency histogram) |

---

## Self-attestation process

1. **Build and run the test suite** against your implementation. The reference suite for .NET 10 + xUnit is at `impl/dotnet/tests/NPS.Daemon.Conformance.Tests/` (planned; tracked alongside NPS Daemon MVP). Python and TypeScript suites are planned for Phase 2.

2. **Produce a results manifest** (JSON) summarizing per-case outcomes:

   ```json
   {
     "profile": "NPS-Node-L1",
     "profile_version": "0.1",
     "iut": {
       "name": "my-daemon",
       "version": "1.0.0",
       "nid": "urn:nps:node:example.com:host-01"
     },
     "peer": {
       "name": "nps-dotnet-reference",
       "version": "1.0.0-alpha.5.2"
     },
     "run": {
       "date": "2026-05-03T00:00:00Z",
       "environment": "linux-x64 / 2 vCPU / 4 GB"
     },
     "cases": [
       { "id": "TC-N1-NCP-01", "result": "pass" },
       { "id": "TC-N1-NCP-02", "result": "pass" }
     ],
     "summary": { "pass": 21, "fail": 0, "skip": 0, "na": 0 }
   }
   ```

   Certification is granted when all 21 cases are `pass` or `na`.

3. **Copy the attestation template** from `spec/services/conformance/NPS-NODE-L1-CERTIFIED.md` in [NPS-Release](https://github.com/labacacia/NPS-Release/tree/main/spec/services/conformance/) to your repository root.

4. **Fill in the template** with your IUT details and embed the results manifest.

5. **Sign the attestation block** with your implementation's root key (see Signing section below).

6. Publish the filled-in `NPS-NODE-L1-CERTIFIED.md` at your repository root as a public claim of compliance.

Self-certification is sufficient for L1 and L2 at this release. Third-party certification via NPS Cloud CA is targeted for L3 in 2027 Q1+.

---

## Signing canonicalization: RFC 8785 JCS

The signed attestation block uses **RFC 8785 JSON Canonicalization Scheme (JCS)** — the same canonicalization used throughout NPS for IdentFrames, AnnounceFrames, and reputation log entries.

JCS rules that matter for the attestation:
- **Encoding**: UTF-8 throughout.
- **Unicode escaping**: control characters (`U+0000`–`U+001F`) MUST be escaped; other characters SHOULD NOT be escaped.
- **Numbers**: IEEE 754 double-precision; no trailing zeros after the decimal point; exponent notation for very large or very small values.
- **Key ordering**: object keys are sorted lexicographically by Unicode code point (ascending).
- **Signature field**: omit the `signature` field from the object before canonicalizing and signing, then attach the resulting signature as the `signature` field.

Sign with the IUT's root Ed25519 private key. The signature value MUST be encoded as `ed25519:<base64url(sig-bytes)>`.

---

## Cross-profile: AaaS L2-08 implies Node-Profile L1

The cross-profile contract in full:

| AaaS claim | Node-Profile obligation |
|-----------|------------------------|
| Claiming **any AaaS level** | No Node-Profile obligation (profiles are orthogonal by default) |
| Claiming **AaaS L2-08** (topology queries) | MUST satisfy **Node-Profile L1** for the host running the Anchor |
| Claiming **AaaS L2-08** with an **active member registry** | SHOULD satisfy **Node-Profile L2** |

This means: if your Anchor exposes `topology.snapshot` and `topology.stream`, your host daemon must have passed the 21 L1 test cases. If your Anchor is backed by a live member registry (via `nps-registry` or equivalent), you are held to the L2 interactive standard.

---

## Where to find the templates

All conformance documents and attestation templates are in the `spec/services/conformance/` directory of the [NPS-Release repository](https://github.com/labacacia/NPS-Release/tree/main/spec/services/conformance/):

| File | Purpose |
|------|---------|
| `NPS-Node-L1.md` | L1 conformance suite (21 `TC-N1-*` test cases, paired-peer methodology, results manifest schema) |
| `NPS-NODE-L1-CERTIFIED.md` | Self-declaration template; copy to your repository root when all 21 cases pass |

L2 and L3 suites and templates will be published when the corresponding requirement IDs are finalized (see NPS-Roadmap Phase 2 and Phase 3).

---

## See also

- [Operator AaaS Profile](Operator-AaaS-Profile) — AaaS compliance levels and the L2-08 / topology:read cross-profile contract
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node) — how to implement an Anchor Node using the NPS SDK

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
