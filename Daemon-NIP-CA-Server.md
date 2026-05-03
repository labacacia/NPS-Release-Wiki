# Daemon: nip-ca-server

> **Audience:** Operators (running an OSS CA for NID issuance)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nip-ca-server` — open-source NIP CA server. Reference implementation of the NIP CA role; issues NIDs to anyone (per its admission policy).

## What this page should contain

- Purpose: OSS reference NIP CA
- Source: `NPS-Dev/tools/nip-ca-server/` (separate from `tools/daemons/`)
- Distribution: `labacacia/nip-ca-server` (PUBLIC)
- Required env vars
- /health shape
- Endpoints: NID issuance, registration, ACME (RFC-0002 — currently EXPERIMENTAL)
- The 5-language port story (per `tools/nip-ca-server/example/` cross-language ports)
- SQLite-backed store (`SqliteNipCaStore`) and pluggable `INipCaStore` injection (added in NPS-Dev #18, #19)
- Relationship to nps-cloud-ca (private cousin)
- Admission / vetting story (impl-defined)

## Source material to draw from

- `NPS-Dev/tools/nip-ca-server/` (full source + examples in 5 languages)
- `spec/NPS-3-NIP.md` §5 (NID issuance)
- `spec/rfcs/NPS-RFC-0002-x509-acme-nid-certs.md` (EXPERIMENTAL ACME path)
- Distribution: `labacacia/nip-ca-server`

## Cross-links

- [Daemon NPS-Cloud-CA](Daemon-NPS-Cloud-CA) (private cousin)
- [Protocol NIP](Protocol-NIP)
- [SDK Identity and Authentication](SDK-Identity-and-Authentication)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
