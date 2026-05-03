# Reference: Cognon (CGN) Budget

> **Audience:** Anyone implementing or consuming token-cost accounting
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

The CGN cost model — formerly NPT (renamed in alpha.5.2). How costs are estimated, declared on the wire, enforced.

## What this page should contain

- Why "Cognon (CGN)" replaced "NPS Token (NPT)" — alpha.5.2 rename rationale
- The `cgn_est` wire field (was `estimated_npt` — full rename, no alias retained at suite level)
- Declaration: where CGN limits live (NWM, ActionSpec, request headers `X-NWP-Budget` / `X-NWP-Tokens`)
- Streaming policy (`token-budget.md` §7): per-batch enforcement vs push-stream agent-side control
- Migration note: clients pinned to alpha.4 still emit `estimated_npt` and will be silently rejected by alpha.5.2 servers
- Sub-section: "Why we use Cognon" — the unit's intent (compute equivalence, not raw tokens)

## Source material to draw from

- `spec/token-budget.md` (currently v0.3)
- The alpha.5.2 CHANGELOG entry on the rename

## Cross-links

- [Protocol NWP](Protocol-NWP) (where the wire field appears)
- [Protocol NOP](Protocol-NOP) (DAG-level CGN flow)
- [SDK Common Patterns](SDK-Common-Patterns) (migration guidance)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
