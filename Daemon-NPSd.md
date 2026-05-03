# Daemon: npsd

> **Audience:** Operators
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`npsd` is the main NPS orchestration runtime daemon. Page covers purpose, deployment, /health, scaling.

## What this page should contain

- Purpose: orchestration runtime that hosts node implementations
- Source location in monorepo: `NPS-Dev/tools/daemons/npsd/`
- Distribution: bundled into `labacacia/nps-daemons` via `sync-nps-daemons.sh`
- docker-compose service definition (from `bundle-overlay/docker-compose.yml`)
- Required env vars (table)
- Optional env vars
- /health response shape (JSON example)
- Exposed ports
- Scaling notes (single instance vs HA)
- Common operational issues

## Source material to draw from

- `NPS-Dev/tools/daemons/npsd/` (Program.cs, README, CHANGELOG, csproj source + publish-overlay)
- `NPS-Dev/tools/daemons/bundle-overlay/docker-compose.yml` (npsd service)
- Distribution repo: `labacacia/nps-daemons` after sync

## Cross-links

- [Operator Daemons Reference](Operator-Daemons-Reference)
- [Operator Quickstart Bundle](Operator-Quickstart-Bundle)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
