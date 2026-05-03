# Repository Topology

> **Audience:** Contributors + curious newcomers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Map of all 17 NPS repos: who owns what, what flows where, what's source-of-truth and what's distribution.

## What this page should contain

- A diagram (mermaid OK) showing NPS-Dev as the hub, with arrows out to NPS-Release, 6 SDKs, 3 ingress, 7 daemons distribution, examples, orchestrator
- Per-repo table: name, role (truth / distribution / consumer), org (labacacia / orilynn-studio / innolotus), public/private
- The 4 stub repos (Studio, NWP-Manager, sdk-cpp, sdk-php) and their tracking convention
- Sync-script mapping: which sync-*.sh produces which distribution repo
- The version.yaml repo list as the canonical inventory

## Source material to draw from

- `NPS-Dev/nps-repo-list.md`
- `NPS-Release/version.yaml`
- `NPS-Dev/tools/release/sync-*.sh` headers (each declares its source + target)

## Cross-links

- [Release Process](Release-Process)
- [Contributing Guide](Contributing-Guide)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
