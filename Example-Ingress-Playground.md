# Example: ingress-playground

> **Audience:** Developers wanting to see end-to-end ingress translation in action
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`ingress-playground` (formerly `bridge-playground`) — runnable example that wires MCP, A2A, and gRPC ingresses into one NPS environment, demonstrating non-NPS-protocol → NPS translation.

## What this page should contain

- What this example demonstrates: 3 ingress protocols (MCP, A2A, gRPC) translating into NPS
- Directory layout in `NPS-examples/ingress-playground/`
- How to run it (`docker compose up` or dotnet run)
- The 3 ingress packages it consumes: `LabAcacia.McpIngress`, `LabAcacia.A2aIngress`, `LabAcacia.GrpcIngress` (formerly `*Bridge` — renamed in alpha.3 by CR-0001)
- The gRPC proto: `labacacia.grpc_ingress.v1.NwpIngress` (renamed from `grpc_bridge.v1`)
- Walk through one round-trip: external request → ingress → NPS frame → backend
- Expected output snapshots (note: snapshots must be refreshed at each suite version)

## Source material to draw from

- `labacacia/NPS-examples/ingress-playground/` (entire directory)
- `labacacia/NPS-examples/README.md` for overview
- The 3 ingress repos: `NPS-mcp-ingress`, `NPS-a2a-ingress`, `NPS-grpc-ingress`

## Cross-links

- [SDK Building a Bridge Node](SDK-Building-a-Bridge-Node)
- [Example Cross-SDK Interop](Example-Cross-SDK-Interop)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
