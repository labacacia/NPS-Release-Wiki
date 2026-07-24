# Specs Index

**Status:** ✅ Content complete — v1.0.0-alpha.16

This page is an index of all NPS protocol specifications, shared reference documents, RFCs, and CRs. It is a navigation aid only.

> **The `spec/` directory in NPS-Release is the normative source of truth.** This wiki page never overrides the spec. If this page and a spec document disagree, the spec document wins.

The canonical spec files are in the `spec/` directory of [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) (distribution) and their source in [`labacacia/NPS-Dev`](https://github.com/labacacia/NPS-Dev/tree/main/spec) (authoring). Both are identical for released versions.

---

## Protocol Specifications

| Spec | Version | Status | Date | What it covers |
|------|---------|--------|------|----------------|
| [NPS-0 Overview](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-0-Overview.md) | v0.4 | Proposed | 2026-04-19 | Suite architecture overview, design principles, frame namespace, encoding tiers, node types, security overview, versioning policy, relationship to existing protocols |
| [NPS-1 NCP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-1-NCP.md) | v0.9 | Proposed | 2026-06-27 | Wire format, frame structure, encoding tiers (JSON Tier-1 / MsgPack Tier-2 / BinaryVector Tier-3 `binary_vector.v1`, v0.9), transport modes (HTTP / native), preamble (RFC-0001), `max_concurrent_streams` negotiation + QUIC stream mapping + rekeying (v0.7), `NopFrame` (0x07) keepalive + `ping_interval_ms` (v0.8), native-mode transport (RFC-0006), semantic compression via AnchorFrame |
| [NPS-2 NWP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-2-NWP.md) | v0.17 | Proposed | 2026-07-05 | Agent query/action protocol, node types (Memory / Action / Complex / Anchor / Bridge), Neural Web Manifest (NWM), graph traversal (§11), topology queries (CR-0002), `SubscribeFrame` formal spec (§13, CR-0006), NWM `manifest_version` / `manifest_updated_at` / `X-NWM-Version` (v0.14), Bridge Node conformance + `bridge_target` vectors (§16, v0.14), `X-NWP-Depth` / `X-NWP-Trace` headers, LLM/Thinking Profile `profiles.llm` (§4.2a, v0.16) + `llm.complete` contract (§7.5, v0.15), HTTP binding rejection codes (§9.5, v0.17) |
| [NPS-3 NIP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-3-NIP.md) | v0.11 | Proposed | 2026-07-04 | Neural Identity Protocol — NID format (`urn:nps:...`), CA hierarchy (Root / Org / Agent / Node / Operator), Ed25519 + ECDSA P-256 signatures, assurance levels (RFC-0003), `cert_chain` / `cert_format`, IANA PEN 65715 OID wire-in (CR-0004), group/session NIDs (CR-0003), `ocsp_staple` (v0.9), `node_roles` (v0.10), short-lived/renewable edge-mTLS cert profile (§6.1, v0.10), standard `llm:*` capability strings (v0.11), reputation log client (RFC-0004), revocation |
| [NPS-4 NDP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-4-NDP.md) | v0.9 | Proposed | 2026-05-21 | Neural Discovery Protocol — AnnounceFrame, ResolveFrame, GraphFrame §5 topology-snapshot format (v0.8), §9 federation forwarding + `SecurityProfile` (v0.8), resolution modes (local multicast / DNS TXT / NPS Cloud Registry), `activation_mode` (ephemeral / resident / hybrid), structured `spawn_spec_ref` + `heartbeat_interval_ms` (v0.9), `node_roles` / `cluster_anchor` / `bridge_protocols` fields |
| [NPS-5 NOP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-5-NOP.md) | v0.7 | Proposed | 2026-05-21 | Neural Orchestration Protocol — multi-agent task dispatch, DAG task flows (TaskFrame 0x40), AlignStream (0x43, supersedes deprecated AlignFrame 0x05) with ack/NAK + aggregate strategies (v0.6), K-of-N sync barriers, webhook HMAC signing, saga compensation (v0.6), `result_ttl_seconds` (v0.7), CR-0007 L3 runtime integration, delegation chain (max depth 3), OpenTelemetry distributed tracing |

### Protocol Dependency Graph

```
NCP (NPS-1)
  └── NWP (NPS-2)
        └── NIP (NPS-3)
              └── NDP (NPS-4)

NCP + NWP + NIP ──► NOP (NPS-5)
```

---

## Shared Reference Documents

| Document | Version | What it covers |
|----------|---------|----------------|
| `spec/error-codes.md` | v1.5 | Unified error code registry — all `{PROTOCOL}-{CATEGORY}-{DETAIL}` codes with NPS status code mappings. The authoritative list for all protocol layers. |
| `spec/status-codes.md` | v0.4 | NPS native status code family and HTTP status code mappings. Used in both native mode and HTTP mode responses. |
| `spec/token-budget.md` | v0.7 | Cognon (CGN) budget specification — the standardized token-unit formerly called NPT. Two profiles: CGN-Estimate (sampling/fallback permitted) and CGN-Billing (`verified_tokenizer`, NID-signed records); §4.2 headers `X-NWP-Tokens-Profile` / `X-NWP-Billing-Record` / `X-NWP-Billing-Tokenizer-Tier`. Tokenizer resolution chain: explicit declaration → auto-match → UTF-8/4 fallback. |
| `spec/frame-registry.yaml` | v0.13 | Machine-readable frame type registry. Consumed by CI (`check-source-of-truth.py`). The canonical namespace for all `0x01`–`0xFF` frame type codes. New frame types MUST be registered here before merging. |
| `spec/transport-profile.md` | v0.1 | Native-mode TCP/QUIC transport profile (RFC-0006) — length-prefix framing and the `NcpNativeClient` / `NcpServer` / `NcpSession` reference model. |
| `spec/NPS-Roadmap.md` | v0.7 | Phase 0–4 roadmap. Phase 0 = foundation; Phase 1 = reference implementation (.NET); Phase 2 = SDK expansion; Phase 3 = production hardening; Phase 4 = ecosystem. |

---

## RFCs

Source: `spec/rfcs/` in NPS-Dev.

| RFC | Title | Status | Alpha version landed |
|-----|-------|--------|---------------------|
| [RFC-0001](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0001-ncp-connection-preamble.md) | Add NCP connection preamble for native-mode traffic identification | Accepted (Phase 1 active) | alpha.3 |
| [RFC-0002](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0002-x509-acme-nid-certs.md) | Adopt X.509 + ACME for NID certificates | Proposed — EXPERIMENTAL (blocked on IANA PEN receipt) | — |
| [RFC-0003](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0003-agent-identity-assurance-levels.md) | Three-tier Agent identity assurance levels for anti-scraping / trust gating | Accepted, Phase 3 gated (21-day notice required before activation) | Phase 1–2 in alpha.5 |
| [RFC-0004](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0004-nid-reputation-log.md) | Append-only NID reputation log (Certificate Transparency for Agents) | Accepted, Phase 3 active | Phase 3 landed in alpha.5 |
| [RFC-0005](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0005-reputation-policy-enforcement.md) | Reputation Policy Enforcement | Draft | — |
| [RFC-0006](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0006-ncp-native-transport.md) | NCP native-mode transport (TCP length-prefix framing) | Draft | Phase 1 in alpha.11 (.NET) |

See [RFC Process](RFC-Process) for the full lifecycle description and how to open a new RFC.

---

## CRs (Change Requests)

Source: `spec/cr/` in NPS-Dev. CRs are pre-1.0 planning artifacts; after v1.0.0 all spec changes use the RFC process.

| CR | Title | Status | Alpha version landed |
|----|-------|--------|---------------------|
| [CR-0001](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0001-anchor-bridge-split.md) | Split Gateway Node into Anchor Node + Bridge Node | Implemented | alpha.3 (2026-04-26) |
| [CR-0002](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0002-anchor-topology-queries.md) | Standard topology query types for Anchor Node (`topology.snapshot` / `topology.stream`) | Implemented | alpha.4 (2026-04-27) |
| [CR-0003](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0003-orchestrator-group-session-nids.md) | Group / session NIDs (`group-` / `session-` prefixes, `IdentFrame.lineage`) | Implemented | NIP v0.8 |
| [CR-0004](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0004-pen-wirein.md) | IANA PEN 65715 OID wire-in (replaces provisional `…99999` arc) | Implemented | NIP v0.8 (PEN assigned 2026-05-08) |
| [CR-0005](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0005-nip-ca-ra-model.md) | NIP-CA registration-authority model (bootstrap tokens, pending registrations) | Implemented | NIP v0.8 |
| [CR-0006](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0006-subscribe-frame.md) | SubscribeFrame formal specification (NWP §13) | Accepted (2026-05-28) | NWP v0.13 |
| [CR-0007](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0007-nop-l3-runtime-integration.md) | NOP L3 runtime integration (nps-runner lease) | Accepted | NOP v0.7 |
| [CR-0008](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0008-tier3-binary-vector.md) | Tier-3 BinaryVector v1 encoding (`binary_vector.v1`) | Proposed | NCP v0.9 |

See [CR Process](CR-Process) for the full description and authoring guide.

---

## Services Layer Specs

| Document | Version | What it covers |
|----------|---------|----------------|
| `spec/services/NPS-AaaS-Profile.md` | v0.7 | Agent-as-a-Service compliance profile (service side). Defines Anchor Node + Bridge Node compliance levels (L1/L2/L3), Vector Proxy Layer, capability gating. NPS-CR-0001 split the original Gateway Node here. |
| `spec/services/NPS-Node-Profile.md` | v0.1 | Node-side compliance spec. Defines L1/L2/L3 compliance levels and `activation_mode` values (ephemeral / resident / hybrid). Orthogonal to AaaS Profile. |

### Conformance Test Suites

| Document | What it covers |
|----------|----------------|
| `spec/services/conformance/NPS-Node-L1.md` | L1 conformance test suite v0.1 — 21 `TC-N1-*` test cases using paired-peer methodology |
| `spec/services/conformance/NPS-NODE-L1-CERTIFIED.md` | L1 self-declaration template |

---

## Related Pages

- [RFC Process](RFC-Process) — how to open and shepherd an RFC
- [CR Process](CR-Process) — lightweight pre-1.0 change process

---

*Last reviewed at suite version: v1.0.0-alpha.16*
