# Protocol: NCP (Neural Connection Protocol)

> **Audience:** SDK developers + protocol designers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Transport layer for the suite. Covers framing, native vs HTTP modes, the 8-byte preamble, and how upper-layer protocols ride on top.

## What this page should contain

- 1-paragraph "what NCP is" intro
- Frame format diagram (1-byte type + flags + length + payload)
- The 8-byte `NPS/1.0\n` preamble — why it exists (RFC-0001), how to handle it
- Native mode vs HTTP mode side-by-side comparison
- Frame-type byte ranges (0x01-0x0F NCP, 0x10-0x1F NWP, etc.) — link to [Reference: Frame Registry](Reference-Frame-Registry)
- ErrorFrame (0xFE) — how upper layers surface errors
- E2E encryption section (NCP §9)
- Common gotchas: extended-header (EXT=1) ambiguity, HTTP-mode body framing

## Source material to draw from

- `spec/NPS-1-NCP.md` (canonical)
- `spec/rfcs/NPS-RFC-0001-ncp-connection-preamble.md` for the preamble rationale
- `spec/frame-registry.yaml`

## Cross-links

- [Protocol Stack Architecture](Protocol-Stack-Architecture)
- [Reference: Frame Registry](Reference-Frame-Registry)
- [SDK Common Patterns](SDK-Common-Patterns)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
