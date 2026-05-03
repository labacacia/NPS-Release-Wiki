# SDK Tutorial: Building a Bridge Node

> **Audience:** Developers (writing protocol translators between NPS and non-NPS systems like MCP, A2A, gRPC)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

How to implement an NPS Bridge Node: declaring `bridge_protocols`, the `bridge_target` parameter, conformance expectations.

## What this page should contain

- What a Bridge Node is and isn't (translation, not gateway — see CR-0001)
- Declaring `bridge_protocols` in NWM and AnnounceFrame
- The `bridge_target` parameter — current schema is implementation-defined (CR-0001 §3.2 — flag this as not yet standardized)
- Reference implementations: `labacacia/NPS-mcp-ingress`, `labacacia/NPS-a2a-ingress`, `labacacia/NPS-grpc-ingress`
- How to map non-NPS errors into the NPS error namespace
- Note: legacy `node_type: "gateway"` MUST be rejected (`NDP-ANNOUNCE-ROLE-REMOVED`)

## Source material to draw from

- `spec/cr/NPS-CR-0001-anchor-bridge-split.md`
- The 3 ingress repos as living reference implementations

## Cross-links

- [Protocol NDP](Protocol-NDP)
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
