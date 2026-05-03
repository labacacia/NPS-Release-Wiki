# Example: cross-sdk-interop

> **Audience:** Developers + SDK maintainers (cross-language parity verification)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`cross-sdk-interop` — runnable cross-language tests verifying that all 6 SDKs handle wire-level details identically.

## What this page should contain

- Purpose: catch behavioral drift between SDKs that the per-SDK tests can't (e.g., serialization byte-for-byte parity, edge-case input handling)
- Test structure
- The `assurance-level-parity.sh` test (added alpha.5.2 follow-up) — verifies `fromWire("")` → ANONYMOUS in all 4 of Python, TS, Java, Go
- How to add a new cross-SDK test (recommended pattern)
- CI integration (when this runs, what failures look like)
- Failure case study: the alpha.5 incident where Python/TS got an empty-string fix but Go/Rust didn't (now this kind of test would have caught it)

## Source material to draw from

- `labacacia/NPS-examples/cross-sdk-interop/`
- `assurance-level-parity.sh` as the worked example
- The 6 SDK distribution repos as test targets

## Cross-links

- [SDK Quickstart](SDK-Quickstart)
- [SDK Identity and Authentication](SDK-Identity-and-Authentication)
- [Release Process](Release-Process)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
