# Daemon: nps-registry

> **Audience:** Operators
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nps-registry` — node registry daemon. Maintains the cluster member registry that Anchor Nodes query for topology.

## What this page should contain

- Purpose
- Source: `NPS-Dev/tools/daemons/nps-registry/`
- Distribution: bundled into `labacacia/nps-daemons`
- Storage backend (current: ?, future: ?)
- Required env vars
- /health shape
- Relationship to NWP `topology.snapshot` / `topology.stream` queries (Anchor reads from here)
- Member-registry semantics — required for AaaS L2-08
- Sub-Anchor recursion (CR-0002 OQ-1, optional at L2)

## Source material to draw from

- `NPS-Dev/tools/daemons/nps-registry/`
- `spec/NPS-2-NWP.md` §12 (topology query types)
- `spec/cr/NPS-CR-0002-anchor-topology-queries.md`

## Cross-links

- [Daemon NPSd](Daemon-NPSd)
- [Protocol NWP](Protocol-NWP)
- [Operator AaaS Profile](Operator-AaaS-Profile)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
