# Release Process

> **Audience:** Release shepherds + contributors who watch a release land
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Whole-suite release flow: how a new alpha.N (or alpha.N.M hotfix) ships across all 17 repos. Single source of truth, version oracle, alignment invariants.

## What this page should contain

- The single-source-of-truth model: NPS-Dev is truth; everything is sync'd out
- Single oracle rule: `NPS-Release/version.yaml` `suite_version` is THE machine-readable version. CHANGELOG is the human notification medium, not the oracle.
- No-per-protocol-versions rule: any change → whole-suite bump. Per-package alpha.5 / alpha.5.1 / alpha.5.2 lattice is forbidden (lessons from the alpha.5.2 incident).
- The whole-suite release sequence (15+ steps): impl/<lang> bump → daemon source csproj → publish-overlay → docker-compose tag → CHANGELOGs → spec → version.yaml → sync scripts → CI green → tag.
- Per-tool release flow: see [`NPS-Dev/docs/release-process.md`](https://github.com/labacacia/NPS-Dev/blob/main/docs/release-process.md) (the existing per-tool SOP)
- Pre-flight alignment checklist (mirror the table the docs-update prompt asked for)
- The drift-prevention CI: `check-version-sync.py` + `check-source-of-truth.py` (Assertions A–E)
- Past incidents: alpha.5.2 hotfix lattice (now guarded by Assertion B/C/D)
- Hotfix flow: minor patch releases (e.g. alpha.5.1, alpha.5.2) still bump the entire suite together

## Source material to draw from

- `NPS-Dev/docs/release-process.md` (per-tool flow)
- `NPS-Dev/tools/release/sync-*.sh` (sync scripts)
- `NPS-Dev/tools/scripts/check-version-sync.py` + `check-source-of-truth.py`
- `NPS-Release/version.yaml` (the oracle)
- Past audit memos in `NPS-Dev/log/audits/` (if maintained)

## Cross-links

- [Repository Topology](Repository-Topology)
- [Contributing Guide](Contributing-Guide)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
