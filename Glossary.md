# Glossary

> **Audience:** Anyone (reference)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Definitions of every domain term used across the suite. Alphabetical. Each term: one-sentence definition + a "defined in" pointer to the canonical spec section.

## What this page should contain

- Definitions for: Agent, Anchor Node, Bridge Node, Capability, CGN (Cognon), Complex Node, DAG (orchestration), DiffFrame, Frame, NCP, NDP, NID (Neural ID), NIP, NOP, Node Roles, NWM (Neural Web Manifest), NWP, NPT (legacy alias for CGN — note deprecation), Reputation Log, STH (Signed Tree Head), Suite Version, Topology Stream
- Each entry format: `**Term** — one-sentence definition. ([defined in spec/X.md §Y](link))`
- A "Renamed in alpha.5/5.2" sub-section listing: `node_kind → node_roles`; `NPT/estimated_npt → CGN/cgn_est`; `Gateway Node → Anchor + Bridge`

## Source material to draw from

- All `spec/NPS-*.md` Terminology sections (§1 in each)
- `spec/cr/NPS-CR-0001-anchor-bridge-split.md` for the Anchor/Bridge naming history
- CHANGELOG entries that introduced renames

## Cross-links

- [What Is NPS](What-Is-NPS)
- Each protocol page links back to its terms here

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
