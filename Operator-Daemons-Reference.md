# Operator Reference: Daemons

> **Audience:** Operators
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

One page that documents every daemon (npsd, nps-runner, nps-gateway, nps-registry, nps-ledger, nps-cloud-ca, nip-ca-server). Each gets a sub-section: purpose, deployment topology, required state, env vars, ports, /health shape, scaling notes.

## What this page should contain

- `## npsd` — the orchestration runtime
- `## nps-runner` — task executor
- `## nps-gateway` — ingress
- `## nps-registry` — node registry
- `## nps-ledger` — reputation log + STH gossip (alpha.5 Phase 3)
- `## nps-cloud-ca` — private NPS Cloud CA
- `## nip-ca-server` — NIP CA Server (OSS, public)
- For each: required env vars (table), exposed ports, /health JSON example, scaling/HA notes
- Cross-link each to its CHANGELOG and source overlay

## Source material to draw from

- `NPS-Dev/tools/daemons/<name>/README.md` for each
- `NPS-Dev/tools/daemons/<name>/Program.cs` for /health shape
- `NPS-Dev/tools/release/sync-*.sh` to know what gets distributed where

## Cross-links

- [Operator Quickstart Bundle](Operator-Quickstart-Bundle)
- [Operator Reputation Log](Operator-Reputation-Log) (deep-dive on nps-ledger)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
