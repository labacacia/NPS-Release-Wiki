# Reference: Status Codes

> **Audience:** Anyone debugging
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

The `NPS-*` status code family — coarser-grained classification that error codes map to.

## What this page should contain

- Mirror `spec/status-codes.md` (currently v0.4)
- Each: code, HTTP equivalent, narrative description
- `NPS-SERVER-UNSUPPORTED` (501) added in alpha.5
- How status codes relate to error codes (one status class can contain many error codes)

## Source material to draw from

- `spec/status-codes.md`

## Cross-links

- [Reference: Error Codes](Reference-Error-Codes)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
