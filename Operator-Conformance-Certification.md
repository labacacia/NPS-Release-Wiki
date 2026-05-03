# Operator: Node Conformance & Certification

> **Audience:** Operators + node implementers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

How to claim Node-Profile L1 / L2 conformance: which test cases your node must pass, how to use the CERTIFIED self-attestation templates, RFC 8785 JCS canonicalization.

## What this page should contain

- Node-Profile vs AaaS-Profile (orthogonal: node = host capability; AaaS = service surface)
- Level 1 surface: 21 `TC-N1-*` test cases
- Level 2 surface: 12 `TC-N2-*` test cases (alpha.5 added 5 negative-path cases)
- Self-attestation: NPS-NODE-L1-CERTIFIED.md and NPS-NODE-L2-CERTIFIED.md templates
- Signing canonicalization: RFC 8785 (JCS) requirements (UTF-8, key ordering, IEEE 754, Unicode escaping)
- Where to publish your CERTIFIED.md (recommendation: in your project root)
- Cross-Profile: claiming AaaS L2-08 implies satisfying Node-Profile L1 (and SHOULD L2 if active member registry)

## Source material to draw from

- `spec/services/conformance/NPS-Node-L1.md`
- `spec/services/conformance/NPS-Node-L2.md`
- `spec/services/conformance/NPS-NODE-L1-CERTIFIED.md` (template)
- `spec/services/conformance/NPS-NODE-L2-CERTIFIED.md` (template)

## Cross-links

- [Operator AaaS Profile](Operator-AaaS-Profile)
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
