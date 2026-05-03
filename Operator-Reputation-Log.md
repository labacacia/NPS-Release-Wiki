# Operator: NID Reputation Log

> **Audience:** Operators (running an nps-ledger instance) + AaaS operators (consuming a log)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Stand up and operate an NID reputation log. Phase 1–3 features and how to wire `reputation_policy` into your NWM.

## What this page should contain

- What the reputation log is and why (CT for Agents)
- Submitting an entry: 12-field shape (RFC-0004 §4.1)
- Severity ladder + incident vocabulary
- Querying entries (Phase 1 HTTP API)
- Phase 2: Merkle tree, STH, inclusion proofs
- Phase 3: STH Gossip Protocol (alpha.5 — `GET /v1/log/gossip/sth`, env vars)
- Setting `reputation_policy` in NWM (RFC-0004 §4.4) — recommended L2-09 default
- Operating an nps-ledger instance: env, scaling, peer config

## Source material to draw from

- `spec/rfcs/NPS-RFC-0004-nid-reputation-log.md`
- `spec/NPS-3-NIP.md` §5.1.2 (entry shape)
- `NPS-Dev/tools/daemons/nps-ledger/` (reference impl)
- `NPS-Dev/tools/daemons/nps-ledger/CHANGELOG.md` for Phase rollout history

## Cross-links

- [Protocol NIP](Protocol-NIP)
- [Operator Daemons Reference](Operator-Daemons-Reference) (nps-ledger section)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
