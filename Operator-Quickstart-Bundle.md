# Operator Quickstart: Daemon Bundle

> **Audience:** Operators (devops / SREs deploying NPS infrastructure)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`docker compose up` to a working NPS deployment with all 4 OSS daemons (npsd / runner / gateway / registry) using the bundle-overlay distribution.

## What this page should contain

- Pull `labacacia/nps-daemons` (the public bundle distribution)
- Walk-through of `docker-compose.yml` — what each service does
- Required env vars (one row per daemon)
- Optional env vars: `NPSLEDGER_PEERS`, `NPSLEDGER_GOSSIP_INTERVAL_S` (alpha.5 STH gossip)
- First-time setup: where data lives, how to back it up
- `/health` checks for each daemon
- Upgrade procedure (pin to suite version per [version.yaml](https://github.com/labacacia/NPS-Release/blob/main/version.yaml))
- Common bring-up errors and the gotcha when host firewalls block discovery

## Source material to draw from

- `NPS-Dev/tools/daemons/bundle-overlay/docker-compose.yml`
- Each daemon's local README under `tools/daemons/<name>/`
- `NPS-Dev/docs/release-process.md` for the upstream sync model

## Cross-links

- [Operator Daemons Reference](Operator-Daemons-Reference)
- [Operator AaaS Profile](Operator-AaaS-Profile)
- [Operator Reputation Log](Operator-Reputation-Log)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
