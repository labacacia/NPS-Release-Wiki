# Protocol: NDP (Neural Discovery Protocol)

> **Audience:** SDK developers + operators
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Discovery layer: how nodes announce themselves and how agents/nodes find each other. DNS TXT records, AnnounceFrame, GraphFrame.

## What this page should contain

- DNS TXT discovery — the `v=nps1` key/value form (alpha.5 explanation note)
- AnnounceFrame (NDP §3.1) — its fields including `node_roles` (renamed from `node_kind` in alpha.5; alias accepted through alpha.5)
- The `activation_mode` field (`ephemeral` / `resident` / `hybrid`) and `activation_endpoint` semantics
- GraphFrame and how the topology view propagates
- Relationship to NWP `topology.snapshot` / `topology.stream` (NWP §12)
- Bridge node `bridge_protocols` declaration (CR-0001)
- Errors: `NDP-ANNOUNCE-ROLE-REMOVED`, `NDP-ANNOUNCE-ROLE-UNKNOWN` (alpha.5)

## Source material to draw from

- `spec/NPS-4-NDP.md` (currently v0.6)
- `spec/cr/NPS-CR-0001-anchor-bridge-split.md`
- `spec/cr/NPS-CR-0002-anchor-topology-queries.md` (NDP-NWP topology relationship)

## Cross-links

- [Protocol NWP](Protocol-NWP)
- [SDK Building a Bridge Node](SDK-Building-a-Bridge-Node)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
