# Daemon: bundle-overlay

> **Audience:** Operators + release shepherds
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

The `bundle-overlay` is not a runtime daemon but the meta-package that produces the public `labacacia/nps-daemons` distribution. Documents what it bundles, how its docker-compose is structured, and image-tag pinning.

## What this page should contain

- Purpose: assembly + distribution of 4 OSS daemons (npsd, runner, gateway, registry) as one git repo + docker-compose
- Source: `NPS-Dev/tools/daemons/bundle-overlay/`
- Distribution: roots of `labacacia/nps-daemons` after `sync-nps-daemons.sh` runs
- The docker-compose.yml: one service per daemon, image tags pinned to suite_version
- Image-tag-vs-suite_version invariant (now CI-checked by Assertion C)
- Supplementary CHANGELOG that aggregates the 4 daemon CHANGELOGs
- Where it lives in the publish-overlay model (vs per-daemon publish-overlay/)

## Source material to draw from

- `NPS-Dev/tools/daemons/bundle-overlay/` (Dockerfile, docker-compose.yml, README, CHANGELOG, etc.)
- `NPS-Dev/tools/release/sync-nps-daemons.sh` (the sync script that uses this)
- `NPS-Dev/docs/release-process.md` for the bundle-vs-single-repo decision

## Cross-links

- [Operator Quickstart Bundle](Operator-Quickstart-Bundle)
- [Operator Daemons Reference](Operator-Daemons-Reference)
- [Release Process](Release-Process)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
