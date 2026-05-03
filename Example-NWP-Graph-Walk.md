# Example: nwp-graph-walk

> **Audience:** Developers learning NWP Complex Node graph traversal
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nwp-graph-walk` — example showing NWP Complex Node graph traversal, depth control, and aggregation queries.

## What this page should contain

- Purpose: demonstrate Complex Node behavior (NWP §11)
- Directory layout in `NPS-examples/nwp-graph-walk/`
- How to run
- What it demonstrates: graph depth control (default 1, max 5), aggregation queries (NWP §6.7), filter syntax
- Expected output
- Snapshot refresh expectation per suite version

## Source material to draw from

- `labacacia/NPS-examples/nwp-graph-walk/`
- `spec/NPS-2-NWP.md` §11 (Complex Node), §6.7 (Aggregation Queries)

## Cross-links

- [Protocol NWP](Protocol-NWP)
- [SDK Common Patterns](SDK-Common-Patterns)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
