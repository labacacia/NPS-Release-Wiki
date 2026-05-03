# Protocol: NIP (Neural Identity Protocol)

> **Audience:** SDK developers + operators (CA + reputation log) + protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Identity layer: NID issuance, IdentFrame, capability registry, three-tier assurance levels, and the reputation log.

## What this page should contain

- NID format and lifecycle
- IdentFrame (0x20 range) — what it carries, how receivers verify
- Capability registry — including the alpha.5-added `topology:read`
- Three-tier assurance levels (`anonymous` / `attested` / `verified`) — RFC-0003
- Forward-compatibility rule for unknown future levels (`NIP-ASSURANCE-UNKNOWN`)
- The X.509 / ACME path (RFC-0002, EXPERIMENTAL — flag the provisional OID)
- Reputation log entry shape (12 fields, RFC-0004)
- Phase-3 STH gossip protocol (alpha.5 addition)
- Authentication errors: `NIP-CERT-*`, `NIP-ASSURANCE-*`, `NIP-REPUTATION-*`

## Source material to draw from

- `spec/NPS-3-NIP.md` (currently v0.6)
- `spec/rfcs/NPS-RFC-0002-x509-acme-nid-certs.md` (Draft — EXPERIMENTAL)
- `spec/rfcs/NPS-RFC-0003-agent-identity-assurance-levels.md` (Accepted, Phase 1–2; flip to Phase 3 gated)
- `spec/rfcs/NPS-RFC-0004-nid-reputation-log.md` (Accepted, Phase 3 active in alpha.5)

## Cross-links

- [Operator Reputation Log](Operator-Reputation-Log)
- [SDK Identity and Authentication](SDK-Identity-and-Authentication)
- [Reference: Error Codes](Reference-Error-Codes)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
