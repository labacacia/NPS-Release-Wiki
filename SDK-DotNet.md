# SDK: .NET / C#

> **Audience:** Developers using the .NET SDK
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Comprehensive .NET SDK reference: NuGet packages, project structure, entry-point classes, idiomatic patterns, and links to the full xmldoc site.

## What this page should contain

- NuGet packages list (NPS.Core, NPS.NWP, NPS.NWP.Anchor, NPS.NWP.Bridge, NPS.NIP, NPS.NDP, NPS.NOP) with current pin
- Target framework + minimum supported runtime
- DI registration entry points (`AddNps*()` extension methods if any)
- Key classes per protocol (e.g. `AnchorNodeMiddleware`, `AssuranceLevels`, `NipSigner`, `AnchorActionSpec`)
- Wire-field property naming convention: CGN suffix (post-alpha.5.2 rename — `EstimatedCgn`, `BudgetCgn`, `AvailableCgn`)
- Backward-compat aliases on read (e.g. middleware accepting `node_kind` filter through alpha.5; deprecation logged)
- Test count + coverage summary
- Link to per-package CHANGELOG in NPS-sdk-dotnet

## Source material to draw from

- `labacacia/NPS-sdk-dotnet/README.md` + `README.cn.md`
- `NPS-sdk-dotnet/src/<package>/` for entry points
- `NPS-Dev/impl/dotnet/` for the upstream source (per single-source-of-truth rule)
- Per-package CHANGELOG.md inside each csproj dir if present

## Cross-links

- [SDK Quickstart](SDK-Quickstart) (multi-lang overview)
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)
- [SDK Identity and Authentication](SDK-Identity-and-Authentication)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
