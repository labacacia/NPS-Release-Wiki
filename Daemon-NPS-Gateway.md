# Daemon: nps-gateway

> **Audience:** Operators
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nps-gateway` — ingress gateway daemon. Handles HTTP-mode NCP termination and routes upstream.

## What this page should contain

- Purpose: HTTP-mode ingress for the suite
- Source: `NPS-Dev/tools/daemons/nps-gateway/`
- Distribution: bundled into `labacacia/nps-daemons`
- Required env vars (upstream targets, TLS config)
- TLS termination story
- Routing rules (path-based to backing nodes)
- /health shape
- HA / load-balancer placement notes

## Source material to draw from

- `NPS-Dev/tools/daemons/nps-gateway/`
- `spec/NPS-1-NCP.md` §2.2 (HTTP mode)

## Cross-links

- [Daemon NPSd](Daemon-NPSd)
- [Protocol NCP](Protocol-NCP)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
