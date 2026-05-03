# SDK Common Patterns

> **Audience:** Developers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Cross-cutting recipes: retries, idempotency, error handling, streaming, capability negotiation, version pinning.

## What this page should contain

- Retries: which errors are retryable (idempotent ActionSpec) vs not
- Streaming queries and subscriptions: cancellation via SubscribeFrame(action="unsubscribe", stream_id=…)
- Capability negotiation pattern (advertise via IdentFrame, gate via NWM)
- Token-budget consumption: reading `X-NWP-Tokens` from streams
- Wire-field migration: how to handle `estimated_npt` → `cgn_est` boundary if you must straddle alpha.5.2
- Pinning suite_version vs per-package pins (rule: always pin to suite_version; per-package versioning forbidden)

## Source material to draw from

- `spec/NPS-2-NWP.md` §6.6 (streaming termination), §8.3 (subscription flow)
- `spec/token-budget.md` (currently v0.3)
- `NPS-Release/version.yaml`

## Cross-links

- [Reference: Cognon Budget](Reference-Cognon-Budget)
- [Release Process](Release-Process)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
