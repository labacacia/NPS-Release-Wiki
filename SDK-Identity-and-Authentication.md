# SDK How-To: Identity and Authentication

> **Audience:** Developers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Practical guide to obtaining a NID, presenting it via IdentFrame, dealing with assurance levels, and consulting the reputation log.

## What this page should contain

- How to get a NID (current path = NIP CA Server; future = ACME via RFC-0002 once non-experimental)
- Presenting IdentFrame, what to put in `assurance_level`
- Receiver-side: how to verify a NID, when to consult the reputation log
- The `AssuranceLevel.fromWire("")` empty-string handling (cross-SDK parity fix from alpha.5; demonstrates why blank should map to `anonymous`)
- The forward-compatibility rule for unknown future assurance levels
- How to gate sensitive actions with `min_assurance_level` (NWM-level + per-ActionSpec)

## Source material to draw from

- `spec/NPS-3-NIP.md` §5 (NID), §5.1.1 (assurance), §5.1.2 (reputation entry)
- `spec/rfcs/NPS-RFC-0003-agent-identity-assurance-levels.md`
- `spec/rfcs/NPS-RFC-0004-nid-reputation-log.md`
- `NPS-examples/cross-sdk-interop/assurance-level-parity.sh`

## Cross-links

- [Protocol NIP](Protocol-NIP)
- [Operator Reputation Log](Operator-Reputation-Log)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
