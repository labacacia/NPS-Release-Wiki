# Reference: Error Codes

> **Audience:** Anyone debugging an error in the wild
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Every `<DOMAIN>-<NAME>` error code in the suite, with status-code mapping, HTTP equivalent, and "what to do about it" guidance.

## What this page should contain

- Mirror the structure of `spec/error-codes.md` (currently v1.2)
- For each code: name, mapped status code (`NPS-*`), HTTP, source of definition, guidance for client/server
- Sub-section: "Codes added by alpha.5" (NWP-RESERVED-TYPE-UNSUPPORTED, NDP-ANNOUNCE-ROLE-{REMOVED,UNKNOWN}, NWP-MANIFEST-NODE-TYPE-{REMOVED,UNKNOWN}, NIP-CERT-*, NIP-REPUTATION-GOSSIP-*, NWP-TOPOLOGY-*)
- Sub-section: "Disambiguation" — pairs of codes that are easy to confuse (`NWP-ACTION-NOT-FOUND` vs `NWP-RESERVED-TYPE-UNSUPPORTED`)

## Source material to draw from

- `spec/error-codes.md` (currently v1.2 — single canonical registry)

## Cross-links

- [Reference: Status Codes](Reference-Status-Codes)
- Each protocol page links here for its code family

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
