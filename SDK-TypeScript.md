# SDK: TypeScript

> **Audience:** Developers using the TypeScript SDK (Node.js + browser via fetch)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

TypeScript/JavaScript SDK reference: npm package, module exports, fetch-API usage, TS strictness compat.

## What this page should contain

- npm package: `@labacacia/nps-sdk` + current pin
- Supported runtimes (Node.js LTS, modern browsers via fetch)
- Module structure: `@labacacia/nps-sdk/{nop,nwp,nip,ndp,ncp}`
- The TS 5.9 `as BodyInit` cast on fetch body (committed in c55e741, propagated to NPS-Dev as part of alpha.5.2 cleanup)
- AssuranceLevel `fromWire("")` semantics (returns Anonymous, parity with other SDKs)
- ESM vs CJS exports
- Browser caveats (no NCP native mode, HTTP only)

## Source material to draw from

- `labacacia/NPS-sdk-ts/README.md` + `README.cn.md`
- `NPS-sdk-ts/src/` for module exports
- `NPS-Dev/impl/typescript/` for upstream source
- `NPS-sdk-ts/package.json` for current dependencies and exports map

## Cross-links

- [SDK Quickstart](SDK-Quickstart)
- [Reference: Cognon Budget](Reference-Cognon-Budget)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
