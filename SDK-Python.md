# SDK: Python

> **Audience:** Developers using the Python SDK
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Python SDK reference: PyPI package, module layout, async story, type hints, idiomatic patterns.

## What this page should contain

- PyPI package name + current pin (track suite_version)
- Supported Python versions
- Module layout: `nps_sdk.nip`, `nps_sdk.nwp`, etc.
- Entry-point classes / functions per protocol
- The `AssuranceLevel.from_wire("")` empty-string handling (alpha.5 fix; demonstrates `if not wire:` semantics)
- Async vs sync API surface
- Type hints + mypy strictness expectations
- Test fixtures and where to find them

## Source material to draw from

- `labacacia/NPS-sdk-py/README.md` + `README.cn.md`
- `NPS-sdk-py/nps_sdk/` for module layout
- `NPS-Dev/impl/python/` for upstream source
- `NPS-sdk-py/CHANGELOG.md`

## Cross-links

- [SDK Quickstart](SDK-Quickstart)
- [SDK Identity and Authentication](SDK-Identity-and-Authentication)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
