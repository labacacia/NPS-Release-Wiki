# RFC Process

> **Audience:** Protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

How to propose, draft, and shepherd an RFC for the NPS suite. Status lifecycle, what each phase means, how acceptance works.

## What this page should contain

- When to write an RFC vs a CR (rule of thumb: RFC = new normative behavior across the wire; CR = scoped change to existing surface)
- The template: `spec/rfcs/template.md`
- Status lifecycle: Draft → Proposed → Accepted (Phase 1) → Phase 2/3 active
- The Depends-On chain: how to reference older specs
- How to handle deferred work (Appendix A phasing tables)
- The flag-day rule (≥21-day notice on NPS-Dev Discussions before activating a Phase 3 breaking flip — see RFC-0003 §8.1)
- Examples to read: RFC-0001 (clean Accept), RFC-0002 (long-running EXPERIMENTAL), RFC-0004 (multi-phase rollout)

## Source material to draw from

- `spec/rfcs/README.md`
- `spec/rfcs/template.md`
- All 4 existing RFCs as worked examples

## Cross-links

- [Specs Index](Specs-Index)
- [CR Process](CR-Process)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
