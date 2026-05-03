# Reference: Frame Registry

> **Audience:** Protocol implementers + low-level debuggers
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

Every frame type byte (0x00–0xFF), which protocol owns it, current schema version, what fields it carries.

## What this page should contain

- Mirror `spec/frame-registry.yaml` (currently v0.10)
- Group by protocol: NCP (0x01–0x0F), NWP (0x10–0x1F), NIP, NDP, NOP
- For each frame: byte, name, owning spec § reference, protocol_version it's at
- The reserved range (0xF0–0xFF) and how new sub-protocols claim space
- ErrorFrame (0xFE) — universal error envelope

## Source material to draw from

- `spec/frame-registry.yaml` (the canonical registry)
- Each protocol spec for the field-level schemas

## Cross-links

- [Protocol NCP](Protocol-NCP)
- All [Protocol-*](Protocol-NWP) pages reference this for their frame types

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
