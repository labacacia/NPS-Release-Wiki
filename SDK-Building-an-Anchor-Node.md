# SDK Tutorial: Building an Anchor Node

> **Audience:** Developers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Walkthrough of standing up an Anchor Node: minimal NWM, registering actions, exposing topology query endpoints, gating with `topology:read`.

## What this page should contain

- What an Anchor Node is (link to Glossary)
- Minimal NWM declaration with required fields
- Registering an ActionSpec (with `min_assurance_level` if needed)
- Implementing `topology.snapshot` / `topology.stream` (mandatory at AaaS L2 if maintaining a member registry — L2-08)
- Wiring the `X-NWP-Capabilities` header check for `topology:read` (alpha.5 M6)
- Conformance: which TC-N2-* test cases your impl must pass for Node-Profile L2 (link to Conformance page)
- Anti-patterns: emitting `estimated_npt` (use `cgn_est` instead — alpha.5.2)

## Source material to draw from

- `spec/services/NPS-AaaS-Profile.md` L2-08
- `spec/services/NPS-Node-Profile.md` §4 (Level 2)
- `spec/services/conformance/NPS-Node-L2.md`
- .NET reference impl: `NPS-Dev/impl/dotnet/src/NPS.NWP.Anchor/AnchorNodeMiddleware.cs`

## Cross-links

- [Protocol NWP](Protocol-NWP)
- [Operator AaaS Profile](Operator-AaaS-Profile)
- [Operator Conformance Certification](Operator-Conformance-Certification)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
