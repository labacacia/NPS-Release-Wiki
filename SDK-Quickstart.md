# SDK Quickstart

> **Audience:** Developers building Agents or Nodes against NPS
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Hello-world per language. Each language section: install, minimal client/server, what to read next.

## What this page should contain

- Six sections in this order: .NET, Python, TypeScript, Java, Go, Rust
- Each section: package install command, 30-line client "send IdentFrame + QueryFrame" sample, 30-line server "accept ActionFrame + return CapsFrame" sample
- A short "languages compared" table at the top: maturity, target framework, entry-point package
- A pinning recommendation: pin to the suite_version in [version.yaml](https://github.com/labacacia/NPS-Release/blob/main/version.yaml), bump together with the suite
- Link out to per-task tutorials (Building an Anchor Node, etc.)

## Source material to draw from

- READMEs of each NPS-sdk-<lang> repo (entry-point examples)
- `NPS-Release/docs/sdks.md` (the Pages site SDK page)
- Cross-SDK interop tests in `NPS-examples/cross-sdk-interop/`

## Cross-links

- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)
- [SDK Building a Bridge Node](SDK-Building-a-Bridge-Node)
- [SDK Common Patterns](SDK-Common-Patterns)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
