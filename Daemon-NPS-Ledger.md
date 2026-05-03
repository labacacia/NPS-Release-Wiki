# Daemon: nps-ledger

> **Audience:** Operators (running a reputation log instance) + AaaS operators (peering with one)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nps-ledger` — NID reputation log daemon. Implements RFC-0004 entry storage, querying, Phase-3 STH gossip federation.

## What this page should contain

- Purpose: append-only signed log for NID reputation entries (CT for Agents)
- Source: `NPS-Dev/tools/daemons/nps-ledger/`
- Distribution: `innolotus/nps-ledger` (PRIVATE — NPS Cloud product)
- Required env vars (data dir, signing key path)
- Optional env vars: `NPSLEDGER_PEERS`, `NPSLEDGER_GOSSIP_INTERVAL_S` (alpha.5 Phase 3)
- /health shape (includes `phase`, `gossip_peers`, `gossip_interval_s`)
- Endpoints: `POST /v1/log/entries`, `GET /v1/log/entries`, `GET /v1/log/sth`, `GET /v1/log/proof`, `GET /v1/log/gossip/sth`
- Fork detection + halt behavior (`NIP-REPUTATION-GOSSIP-FORK`)
- Storage backend, retention, scaling

## Source material to draw from

- `NPS-Dev/tools/daemons/nps-ledger/` (full source, includes GossipState.cs, GossipService.cs, Program.cs)
- `spec/rfcs/NPS-RFC-0004-nid-reputation-log.md`
- `spec/NPS-3-NIP.md` §5.1.2 (entry shape)
- `NPS-Dev/tools/daemons/nps-ledger/CHANGELOG.md` (Phase rollout history)

## Cross-links

- [Operator Reputation Log](Operator-Reputation-Log) (operator-side how-to)
- [Protocol NIP](Protocol-NIP)
- [Operator Daemons Reference](Operator-Daemons-Reference)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
