# Operator: AaaS Profile (L1 / L2 / L3)

> **Audience:** Operators (especially AaaS providers — Agent-as-a-Service vendors)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

The three-tier service-side conformance profile. What each level requires, who needs it, how to certify.

## What this page should contain

- The "AaaS" framing: what an Agent-as-a-Service provider is
- Level 1: minimum surface
- Level 2: full Anchor Node behavior (incl. L2-08 topology queries, L2-09 reputation_policy SHOULD)
- Level 3: advanced (placeholder if not yet defined)
- Relationship to Node-Profile (operator-side) — see Conformance page
- Required Depends-On versions per profile level (mirror current AaaS-Profile.md table)

## Source material to draw from

- `spec/services/NPS-AaaS-Profile.md` (currently v0.6)
- `spec/services/NPS-AaaS-Profile.cn.md`

## Cross-links

- [Operator Conformance Certification](Operator-Conformance-Certification)
- [Protocol NWP](Protocol-NWP) §12

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
