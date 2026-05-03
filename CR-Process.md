# CR (Change Request) Process

> **Audience:** Protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Lighter-weight Change Request artifact, parallel to the RFC track. When to use and how.

## What this page should contain

- CR vs RFC: CR is for scoped, near-term changes that don't introduce new surface
- The CR README: `spec/cr/README.md`
- Template (if exists; otherwise model on CR-0001 / CR-0002)
- Examples: CR-0001 (Anchor/Bridge split — implemented alpha.3), CR-0002 (topology queries — implemented alpha.4)
- Hard-rejection vs alias-acceptance patterns: when to use `-REMOVED` error codes vs alias-with-warning
- The "alias accepted through alpha.N" timing rule and how to instrument deprecation telemetry (see Anchor middleware)

## Source material to draw from

- `spec/cr/README.md`
- CR-0001 + CR-0002 as worked examples

## Cross-links

- [RFC Process](RFC-Process)
- [Specs Index](Specs-Index)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
