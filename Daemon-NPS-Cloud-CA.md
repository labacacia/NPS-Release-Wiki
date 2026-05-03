# Daemon: nps-cloud-ca

> **Audience:** NPS Cloud operators (private — innolotus org)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nps-cloud-ca` — private NPS Cloud CA daemon. Issues NIDs for NPS Cloud subscribers. Internal product, not OSS.

## What this page should contain

- Purpose: private CA for NPS Cloud NID issuance
- Source: `NPS-Dev/tools/daemons/nps-cloud-ca/`
- Distribution: `innolotus/nps-cloud-ca` (PRIVATE)
- Required env vars
- /health shape
- Operator API: NID issuance, revocation, audit
- Difference from `nip-ca-server` (the OSS public CA): nps-cloud-ca is multi-tenant + billing-aware
- Note: this page documents the OSS-visible interface only; internal-product details are in the private repo

## Source material to draw from

- `NPS-Dev/tools/daemons/nps-cloud-ca/`
- `spec/NPS-3-NIP.md` for the NID format being issued

## Cross-links

- [Daemon NIP-CA-Server](Daemon-NIP-CA-Server) (the OSS counterpart)
- [Protocol NIP](Protocol-NIP)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
