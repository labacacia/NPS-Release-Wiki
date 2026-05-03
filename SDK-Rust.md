# SDK: Rust

> **Audience:** Developers using the Rust SDK
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Rust SDK reference: crates.io coordinates, workspace layout, async runtime requirements.

## What this page should contain

- crates.io coordinates: `nps-core`, `nps-ncp`, `nps-nwp`, `nps-nip`, `nps-ndp`, `nps-nop` + current pin (workspace version)
- MSRV (minimum supported Rust version) per Cargo.toml
- Workspace layout: shows the single `workspace.package.version = "..."` entry that flattens all crates
- The `from_wire("")` returning ANONYMOUS (uses `if wire.is_empty()`)
- Async runtime requirements (tokio? async-std?)
- Feature flags (if any): NCP native, TLS providers, etc.
- crates.io publishing flow (per-crate vs workspace)

## Source material to draw from

- `labacacia/NPS-sdk-rust/README.md` + `README.cn.md`
- `NPS-sdk-rust/{nps-core,nps-ncp,...}/src/` for crate layouts
- `NPS-Dev/impl/rust/` for upstream source
- `NPS-sdk-rust/Cargo.toml` (workspace)

## Cross-links

- [SDK Quickstart](SDK-Quickstart)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
