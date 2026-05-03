# Protocol: NOP (Neural Orchestration Protocol)

> **Audience:** SDK developers + protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Orchestration layer: DAG-based task dispatch, delegation chains with depth limits, scope-bounded sub-tasks.

## What this page should contain

- DAG model — how tasks compose
- DelegateFrame and the 3-level delegation depth rule (Orchestrator → Worker → Sub-Worker; `NOP-DELEGATE-CHAIN-TOO-DEEP`)
- Scope enforcement: how `delegated_scope` is bounded by the parent's scope
- Condition expressions (CEL subset) — variable binding rules
- Async vs sync action invocation (timeouts, idempotency)
- Errors: `NOP-DELEGATE-*`, `NOP-RESOURCE-INSUFFICIENT` (CGN-related)

## Source material to draw from

- `spec/NPS-5-NOP.md` (canonical)
- `spec/error-codes.md` for the NOP-* code family

## Cross-links

- [Protocol NIP](Protocol-NIP) — capability + scope binding
- [Reference: Cognon Budget](Reference-Cognon-Budget) — CGN cost flowing through DAGs

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
