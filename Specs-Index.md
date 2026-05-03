# Specs Index

> **Audience:** Protocol designers + spec contributors
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Index of all normative documents in `spec/`, with current version and status. Navigation hub for the spec tree.

## What this page should contain

- The 5 protocol specs (NPS-0..5) with current versions, dates, brief descriptions
- The 4 RFCs with status (Draft / Accepted / Phase-1/2/3-active)
- The 2 CRs with status
- AaaS-Profile + Node-Profile + 2 conformance suites
- Cross-cutting: error-codes, status-codes, frame-registry, token-budget (Cognon)
- A "how to read a spec" sub-section: section conventions, normative SHOULD/MUST language, Depends-On chains
- Link to the spec governance / template

## Source material to draw from

- `spec/` directory inventory
- `spec/rfcs/README.md` and `spec/cr/README.md`

## Cross-links

- [RFC Process](RFC-Process)
- [CR Process](CR-Process)
- [Protocol Stack Architecture](Protocol-Stack-Architecture)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
