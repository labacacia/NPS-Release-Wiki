# Protocol: NWP (Neural Web Protocol)

> **Audience:** SDK developers + protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

The HTTP-equivalent surface for Agent ↔ Node interaction. Covers NWM, QueryFrame/ActionFrame/SubscribeFrame, and the alpha.5 topology query namespace.

## What this page should contain

- 1-paragraph intro: NWP is to NPS what HTTP is to TCP
- NWM (Neural Web Manifest) — what it declares, where it lives
- The three workhorse frames: QueryFrame (0x10), ActionFrame (0x11), SubscribeFrame (0x12)
- Streaming queries & subscriptions; relationship to DiffFrame (0x02)
- The §12 reserved query namespace and `topology.snapshot` / `topology.stream` (added alpha.4 by CR-0002)
- The `topology:read` capability gate (alpha.5 M6 fix; `X-NWP-Capabilities` header)
- The `min_assurance_level` field at NWM and per-ActionSpec
- The CGN cost model and how `cgn_est` flows through (renamed from `estimated_npt` in alpha.5.2)
- Common errors: `NWP-RESERVED-TYPE-UNSUPPORTED`, `NWP-TOPOLOGY-*`, `NWP-AUTH-ASSURANCE-TOO-LOW`

## Source material to draw from

- `spec/NPS-2-NWP.md` (canonical)
- `spec/cr/NPS-CR-0002-anchor-topology-queries.md`
- `spec/services/NPS-AaaS-Profile.md` for L2-08/L2-09 cross-references
- `spec/error-codes.md` for the NWP-* code family

## Cross-links

- [Protocol NCP](Protocol-NCP)
- [Operator AaaS Profile](Operator-AaaS-Profile)
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)
- [Reference: Cognon Budget](Reference-Cognon-Budget)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
