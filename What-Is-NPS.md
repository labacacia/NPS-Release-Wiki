# What Is NPS

> **Audience:** Newcomers; ≤ 5 minutes to read
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

One-page elevator pitch: what NPS is, what problem it solves, who it's for, and what's in the suite. End with a 3-link "now go to…" choose-your-path.

## What this page should contain

- A 3-paragraph elevator pitch (problem → suite → who uses it)
- A diagram with the 5-protocol stack (NCP at the bottom, NWP/NIP/NDP at L2, NOP at L3)
- One paragraph each on: Anchor Node, Bridge Node, Agent
- A "Compared to MCP / A2A / gRPC" callout — short
- Three "go to next" links: SDK Quickstart, Operator Quickstart, Specs Index

## Source material to draw from

- `spec/NPS-0-Overview.md` (the single most important source for this page)
- `docs/overview.md` from the Pages site
- `README.md` of NPS-Release for the elevator pitch tone

## Cross-links

- [Glossary](Glossary)
- [Protocol Stack Architecture](Protocol-Stack-Architecture)
- [SDK Quickstart](SDK-Quickstart)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
