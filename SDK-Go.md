# SDK: Go

> **Audience:** Developers using the Go SDK
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Go SDK reference: module path, package layout, idiomatic Go patterns.

## What this page should contain

- Module path: `github.com/labacacia/NPS-sdk-go` + current pin
- Supported Go versions (verify in go.mod)
- Package layout: `github.com/labacacia/NPS-sdk-go/{ncp,nwp,nip,ndp,nop}`
- The `AssuranceFromWire("")` returning `AssuranceAnonymous` (uses `if wire == ""`)
- Idiomatic patterns: explicit error returns, context cancellation, channels for streaming
- Note: Go module currently lacks a VERSION constant — issue/follow-up to add one

## Source material to draw from

- `labacacia/NPS-sdk-go/README.md` + `README.cn.md`
- `NPS-sdk-go/{ncp,nwp,nip,ndp,nop}/` for package layout
- `NPS-Dev/impl/go/` for upstream source
- `NPS-sdk-go/go.mod`

## Cross-links

- [SDK Quickstart](SDK-Quickstart)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
