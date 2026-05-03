# Specs Index

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

This page is an index of all NPS protocol specifications, shared reference documents, RFCs, and CRs. It is a navigation aid only.

> **The `spec/` directory in NPS-Release is the normative source of truth.** This wiki page never overrides the spec. If this page and a spec document disagree, the spec document wins.

The canonical spec files are in the `spec/` directory of [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) (distribution) and their source in [`labacacia/NPS-Dev`](https://github.com/labacacia/NPS-Dev/tree/main/spec) (authoring). Both are identical for released versions.

---

## Protocol Specifications

| Spec | Version | Status | Date | What it covers |
|------|---------|--------|------|----------------|
| [NPS-0 Overview](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-0-Overview.md) | v0.3 | Proposed | 2026-04-19 | Suite architecture overview, design principles, frame namespace, encoding tiers, node types, security overview, versioning policy, relationship to existing protocols |
| [NPS-1 NCP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-1-NCP.md) | v0.6 | Proposed | 2026-04-25 | Wire format, frame structure, encoding tiers (JSON Tier-1 / MsgPack Tier-2), transport modes (HTTP / native), preamble (RFC-0001), connection preamble, semantic compression via AnchorFrame |
| [NPS-2 NWP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-2-NWP.md) | v0.10 | Proposed | 2026-05-01 | Agent query/action protocol, node types (Memory / Action / Complex / Anchor / Bridge), Neural Web Manifest (NWM), graph traversal (§11), topology queries (CR-0002), `X-NWP-Depth` / `X-NWP-Trace` headers |
| [NPS-3 NIP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-3-NIP.md) | v0.6 | Proposed | 2026-05-01 | Neural Identity Protocol — NID format (`urn:nps:...`), CA hierarchy (Root / Org / Agent / Node / Operator), Ed25519 + ECDSA P-256 signatures, assurance levels (RFC-0003), reputation log client (RFC-0004), revocation |
| [NPS-4 NDP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-4-NDP.md) | v0.6 | Proposed | 2026-05-01 | Neural Discovery Protocol — AnnounceFrame, ResolveFrame, resolution modes (local multicast / DNS TXT / NPS Cloud Registry), DNS TXT fallback parsing, `activation_mode` (ephemeral / resident / hybrid), `node_roles` / `cluster_anchor` / `bridge_protocols` fields |
| [NPS-5 NOP](https://github.com/labacacia/NPS-Release/blob/main/spec/NPS-5-NOP.md) | v0.4 | Proposed | 2026-04-19 | Neural Orchestration Protocol — multi-agent task dispatch, DAG task flows (TaskFrame 0x40), AlignStream (0x43, supersedes deprecated AlignFrame 0x05), K-of-N sync barriers, cross-agent result sharing, delegation chain (max depth 3), OpenTelemetry distributed tracing |

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
| `spec/error-codes.md` | v1.2 | Unified error code registry — all `{PROTOCOL}-{CATEGORY}-{DETAIL}` codes with NPS status code mappings. The authoritative list for all protocol layers. |
| `spec/status-codes.md` | v0.4 | NPS native status code family and HTTP status code mappings. Used in both native mode and HTTP mode responses. |
| `spec/token-budget.md` | v0.3 | Cognon (CGN) budget specification — the standardized token-unit formerly called NPT. Tokenizer resolution chain: explicit declaration → auto-match → UTF-8/4 fallback. |
| `spec/frame-registry.yaml` | v0.10 | Machine-readable frame type registry. Consumed by CI (`check-source-of-truth.py`). The canonical namespace for all `0x01`–`0xFF` frame type codes. New frame types MUST be registered here before merging. |
| `spec/NPS-Roadmap.md` | v0.3 | Phase 0–4 roadmap. Phase 0 = foundation; Phase 1 = reference implementation (.NET); Phase 2 = SDK expansion; Phase 3 = production hardening; Phase 4 = ecosystem. |

---

## RFCs

Source: `spec/rfcs/` in NPS-Dev.

| RFC | Title | Status | Alpha version landed |
|-----|-------|--------|---------------------|
| [RFC-0001](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0001-ncp-connection-preamble.md) | Add NCP connection preamble for native-mode traffic identification | Accepted (Phase 1 active) | alpha.3 |
| [RFC-0002](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0002-x509-acme-nid-certs.md) | Adopt X.509 + ACME for NID certificates | Proposed — EXPERIMENTAL (blocked on IANA PEN receipt) | — |
| [RFC-0003](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0003-agent-identity-assurance-levels.md) | Three-tier Agent identity assurance levels for anti-scraping / trust gating | Accepted, Phase 3 gated (21-day notice required before activation) | Phase 1–2 in alpha.5 |
| [RFC-0004](https://github.com/labacacia/NPS-Dev/blob/main/spec/rfcs/NPS-RFC-0004-nid-reputation-log.md) | Append-only NID reputation log (Certificate Transparency for Agents) | Accepted, Phase 3 active | Phase 3 landed in alpha.5 |

See [RFC Process](RFC-Process) for the full lifecycle description and how to open a new RFC.

---

## CRs (Change Requests)

Source: `spec/cr/` in NPS-Dev. CRs are pre-1.0 planning artifacts; after v1.0.0 all spec changes use the RFC process.

| CR | Title | Status | Alpha version landed |
|----|-------|--------|---------------------|
| [CR-0001](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0001-anchor-bridge-split.md) | Split Gateway Node into Anchor Node + Bridge Node | Implemented | alpha.3 (2026-04-26) |
| [CR-0002](https://github.com/labacacia/NPS-Dev/blob/main/spec/cr/NPS-CR-0002-anchor-topology-queries.md) | Standard topology query types for Anchor Node (`topology.snapshot` / `topology.stream`) | Implemented | alpha.4 (2026-04-27) |

See [CR Process](CR-Process) for the full description and authoring guide.

---

## Services Layer Specs

| Document | Version | What it covers |
|----------|---------|----------------|
| `spec/services/NPS-AaaS-Profile.md` | v0.6 | Agent-as-a-Service compliance profile (service side). Defines Anchor Node + Bridge Node compliance levels (L1/L2/L3), Vector Proxy Layer, capability gating. NPS-CR-0001 split the original Gateway Node here. |
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

*Last reviewed at suite version: v1.0.0-alpha.5.2*
