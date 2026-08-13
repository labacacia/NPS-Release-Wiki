# Repository Topology

**Status:** ✅ Latest published topology — v1.0.0-alpha.16

This page maps every NPS-related repository, its role, and how it relates to the central source monorepo.

---

## High-Level Diagram

```
labacacia/NPS-Dev  (source monorepo — all authoring happens here)
│
├── spec/ ──────────────────────────────► labacacia/NPS-Release
│   (specs, RFCs, CRs)                    (GitHub Pages + spec distribution)
│
├── impl/dotnet/ ────────────────────────► labacacia/NPS-sdk-dotnet
├── impl/python/ ────────────────────────► labacacia/NPS-sdk-py
├── impl/typescript/ ────────────────────► labacacia/NPS-sdk-ts
├── impl/java/ ──────────────────────────► labacacia/NPS-sdk-java
├── impl/rust/ ──────────────────────────► labacacia/NPS-sdk-rust
├── impl/go/ ────────────────────────────► labacacia/NPS-sdk-go
│
├── tools/daemons/                        ┌─ labacacia/nps-daemons (bundle, public)
│   ├── npsd/          ──────────────────►├── npsd/
│   ├── nps-runner/    ──────────────────►├── nps-runner/
│   ├── nps-ingress/   ──────────────────►├── nps-ingress/
│   ├── nps-registry/  ──────────────────►└── nps-registry/
│   ├── nps-cloud-ca/  ──────────────────► innolotus/nps-cloud-ca (private)
│   └── nps-ledger/    ──────────────────► innolotus/nps-ledger (private)
│
├── tools/nip-ca-server/ ────────────────► labacacia/nip-ca-server
│
├── compat/mcp-ingress/ ─────────────────► labacacia/NPS-mcp-ingress
├── compat/a2a-ingress/ ─────────────────► labacacia/NPS-a2a-ingress
└── compat/grpc-ingress/ ────────────────► labacacia/NPS-grpc-ingress

orilynn-studio/nps-orchestrator  (independent consumer / example — not synced from NPS-Dev)
labacacia/NPS-examples           (curated demos — source in NPS-Dev demos/)
```

All arrows are one-way syncs: NPS-Dev → distribution repo. Distribution repos are never edited directly; their next state comes from the next sync run.

---

## Repository Table

| Repo | Organization | Role | Visibility | Source of Truth |
|------|-------------|------|------------|----------------|
| `NPS-Dev` | labacacia | Source monorepo — all spec authoring, SDK development, tooling | Public | YES — all source lives here |
| `NPS-Release` | labacacia | Spec distribution + GitHub Pages docs site | Public | Specs only (synced from NPS-Dev `spec/`) |
| `NPS-sdk-dotnet` | labacacia | .NET SDK distribution | Public | NO — synced from NPS-Dev `impl/dotnet/` |
| `NPS-sdk-py` | labacacia | Python SDK distribution | Public | NO — synced from NPS-Dev `impl/python/` |
| `NPS-sdk-ts` | labacacia | TypeScript SDK distribution | Public | NO — synced from NPS-Dev `impl/typescript/` |
| `NPS-sdk-java` | labacacia | Java SDK distribution | Public | NO — synced from NPS-Dev `impl/java/` |
| `NPS-sdk-rust` | labacacia | Rust SDK distribution | Public | NO — synced from NPS-Dev `impl/rust/` |
| `NPS-sdk-go` | labacacia | Go SDK distribution | Public | NO — synced from NPS-Dev `impl/go/` |
| `nps-daemons` | labacacia | OSS daemon bundle (npsd + nps-runner + nps-ingress + nps-registry) | Public | NO — synced from NPS-Dev `tools/daemons/` (4 OSS daemons + bundle-overlay) |
| `nip-ca-server` | labacacia | NIP CA Server standalone distribution | Public | NO — synced from NPS-Dev `tools/nip-ca-server/` |
| `NPS-mcp-ingress` | labacacia | MCP Ingress adapter distribution (`LabAcacia.McpIngress`) | Public | NO — synced from NPS-Dev `compat/mcp-ingress/` |
| `NPS-a2a-ingress` | labacacia | A2A Ingress adapter distribution (`LabAcacia.A2aIngress`) | Public | NO — synced from NPS-Dev `compat/a2a-ingress/` |
| `NPS-grpc-ingress` | labacacia | gRPC Ingress adapter distribution (`LabAcacia.GrpcIngress`) | Public | NO — synced from NPS-Dev `compat/grpc-ingress/` |
| `NPS-examples` | labacacia | Curated runnable demos (`nwp-graph-walk`, `ingress-playground`, `cross-sdk-interop`) | Public | NO — mirrors NPS-Dev `demos/`; tagged independently |
| `nps-cloud-ca` | innolotus | NPS Cloud CA daemon (private — 2027 Q1+ target) | **Private** | NO — synced from NPS-Dev `tools/daemons/nps-cloud-ca/` |
| `nps-ledger` | innolotus | K-of-N audit + reputation log daemon | **Private** | NO — synced from NPS-Dev `tools/daemons/nps-ledger/` |
| `nps-orchestrator` | orilynn-studio | Consumer / example orchestrator service | Public | Independent — not synced from NPS-Dev; tracked by `version.yaml` for version parity only |

> **Ingress packages on the suite train.** The three ingress adapter packages (`LabAcacia.McpIngress`, `LabAcacia.A2aIngress`, `LabAcacia.GrpcIngress`) — deferred in the alpha.13 re-cut — are now caught up and publish on the suite train: all three ship at **v1.0.0-alpha.15**, alongside the 11 SDK packages.

---

## Stub Repos (Tracked, Not Yet Released)

These repos exist as public stubs with README placeholder files. CI tracks them in `version.yaml` with `expected: stub` — they are not required to carry a real version but their README must contain a tracking declaration.

| Repo | Organization | Planned purpose |
|------|-------------|-----------------|
| `NPS-Studio` | labacacia | Frame-stream visualizer / debugger (NPS-Dev alpha.6 queue) |
| `NPS-NWP-Manager` | labacacia | Web-based NWM authoring and node management tool — now ships a runnable **v0.1 stub** (`GET /health`, `GET /v1/nodes`) as of alpha.13 |
| `NPS-sdk-cpp` | labacacia | C++ SDK |
| `NPS-sdk-php` | labacacia | PHP SDK |

---

## Sync-Script Mapping

Scripts live in `tools/release/` in NPS-Dev. Each script:
1. `rsync`s the relevant source directory into a fresh clone of the target publish repo
2. Overlays `publish-overlay/` files (csproj, Dockerfile, docker-compose, nuget.config variants that differ from the monorepo flavor)
3. Copies `LICENSE` and `NOTICE` from the monorepo root
4. Commits, tags (idempotent), pushes to GitHub
5. Invokes the Gitee mirror script with labacacia link rewriting

| Script | Source in NPS-Dev | Target repo |
|--------|------------------|------------|
| `sync-nps-daemons.sh` | `tools/daemons/{npsd,nps-runner,nps-ingress,nps-registry}/` + `bundle-overlay/` | `labacacia/nps-daemons` → Gitee mirror |
| `sync-nip-ca-server.sh` | `tools/nip-ca-server/` | `labacacia/nip-ca-server` → Gitee mirror |
| `sync-nps-cloud-ca.sh` | `tools/daemons/nps-cloud-ca/` | `innolotus/nps-cloud-ca` (no Gitee) |
| `sync-nps-ledger.sh` | `tools/daemons/nps-ledger/` | `innolotus/nps-ledger` (no Gitee) |

SDK and ingress sync scripts follow the same pattern but are per-language / per-adapter.

### Publish Overlay Pattern

Each daemon (and the NIP CA Server) has a `publish-overlay/` subdirectory inside the NPS-Dev source tree. The overlay holds files that must differ between the monorepo build and the standalone published build:

- `publish-overlay/*.csproj` — uses `<PackageReference>` (NuGet) instead of `<ProjectReference>` (monorepo-local path)
- `publish-overlay/Dockerfile` — standalone image build
- `publish-overlay/docker-compose.yml` — standalone deployment compose
- `publish-overlay/nuget.config` — points to nuget.org only (no internal feed)

The sync script copies everything from the source directory, then overlays and deletes the `publish-overlay/` subdirectory so it does not appear in the published repo.

---

## Gitee Mirror

All public labacacia repos are mirrored to Gitee (`gitee.com/labacacia/`) via `tools/mirror-to-gitee/sync-to-gitee.sh`. The Gitee mirror rewrites GitHub URLs to Gitee URLs within README and CHANGELOG files. Private innolotus repos are not mirrored.

---

## Related Pages

- [Release Process](Release-Process) — how a release is prepared and synced to distribution repos

---

*Last reviewed for published packages: v1.0.0-alpha.16*

---

*Last reviewed at suite version: v1.0.0-alpha.18 candidate*
