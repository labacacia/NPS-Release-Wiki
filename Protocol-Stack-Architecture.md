# Protocol Stack Architecture

> **Audience:** Newcomers + protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

How the 5 protocols relate, what runs over what, where the layer boundaries are. The architectural "why" — distinct from per-protocol pages which cover "what".

## What this page should contain

- The 5-layer diagram: NCP (transport) → {NWP, NIP, NDP} (L2 parallel) → NOP (L3 orchestration)
- Why NCP has both native and HTTP modes (NCP §2.2)
- Why NIP/NDP are siblings of NWP rather than nested (identity vs discovery vs application)
- Where Anchor / Bridge / Complex / Memory / Action node types fit
- Cross-protocol concerns: error code namespaces, frame-type byte ranges, status codes
- A small section on "how NPS differs from {MCP, A2A, gRPC} layering"

## Source material to draw from

- `spec/NPS-0-Overview.md` §2 / §3
- `spec/NPS-Roadmap.md` for the layered evolution narrative
- `docs/protocols.md` from Pages site
- `spec/frame-registry.yaml` for the frame-type byte-range table

## Cross-links

- [What Is NPS](What-Is-NPS)
- Each [Protocol-*](Protocol-NCP) page
- [Reference: Frame Registry](Reference-Frame-Registry)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
