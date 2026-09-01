# Repository Topology

**Status:** ✅ Latest published topology — v1.0.0-alpha.18

This page maps every NPS-related repository, its role, and how it relates to the central source monorepo.

---

## High-Level Diagram

```
labacacia/NPS-Dev  (source monorepo — all authoring happens here)
│
├── spec/ ──────────────────────────────► labacacia/NPS-Release
│   (specs, RFCs, CRs)                    (GitHub Pages + spec distribution)
│
├── impl/dotnet/ ────────────────────────► labacacia/NPS-SDK-DotNet
├── impl/python/ ────────────────────────► labacacia/NPS-SDK-Python
├── impl/typescript/ ────────────────────► labacacia/NPS-SDK-TypeScript
├── impl/java/ ──────────────────────────► labacacia/NPS-SDK-Java
├── impl/rust/ ──────────────────────────► labacacia/NPS-SDK-Rust
├── impl/go/ ────────────────────────────► labacacia/NPS-SDK-Go
│
├── tools/daemons/                        ┌─ labacacia/NPS-Daemons (bundle, public)
│   ├── npsd/          ──────────────────►├── npsd/
│   ├── nps-runner/    ──────────────────►├── nps-runner/
│   ├── nps-ingress/   ──────────────────►├── nps-ingress/
│   ├── nps-registry/  ──────────────────►└── nps-registry/
│   ├── nps-cloud-ca/  ──────────────────► labacacia/NPS-Cloud-CA (private)
│   └── nps-ledger/    ──────────────────► labacacia/NPS-Ledger (private)
│
├── tools/nip-ca-server/ ────────────────► labacacia/NIP-CA-Server
│
├── compat/mcp-ingress/ ─────────────────► labacacia/NPS-MCP-Ingress
├── compat/a2a-ingress/ ─────────────────► labacacia/NPS-A2A-Ingress
└── compat/grpc-ingress/ ────────────────► labacacia/NPS-gRPC-Ingress

orilynn-studio/nps-orchestrator  (independent consumer / example — not synced from NPS-Dev)
labacacia/NPS-Examples           (curated demos — source in NPS-Dev demos/)
```

All arrows are one-way syncs: NPS-Dev → distribution repo. Distribution repos are never edited directly; their next state comes from the next sync run.

---

## Repository Table

| Repo | Organization | Role | Visibility | Source of Truth |
|------|-------------|------|------------|----------------|
| `NPS-Dev` | labacacia | Source monorepo — all spec authoring, SDK development, tooling | Public | YES — all source lives here |
| `NPS-Release` | labacacia | Spec distribution + GitHub Pages docs site | Public | Specs only (synced from NPS-Dev `spec/`) |
| `NPS-SDK-DotNet` | labacacia | .NET SDK distribution | Public | NO — synced from NPS-Dev `impl/dotnet/` |
| `NPS-SDK-Python` | labacacia | Python SDK distribution | Public | NO — synced from NPS-Dev `impl/python/` |
| `NPS-SDK-TypeScript` | labacacia | TypeScript SDK distribution | Public | NO — synced from NPS-Dev `impl/typescript/` |
| `NPS-SDK-Java` | labacacia | Java SDK distribution | Public | NO — synced from NPS-Dev `impl/java/` |
| `NPS-SDK-Rust` | labacacia | Rust SDK distribution | Public | NO — synced from NPS-Dev `impl/rust/` |
| `NPS-SDK-Go` | labacacia | Go SDK distribution | Public | NO — synced from NPS-Dev `impl/go/` |
| `NPS-Daemons` | labacacia | OSS daemon bundle (npsd + nps-runner + nps-ingress + nps-registry) | Public | NO — synced from NPS-Dev `tools/daemons/` (4 OSS daemons + bundle-overlay) |
| `NIP-CA-Server` | labacacia | NIP CA Server standalone distribution | Public | NO — synced from NPS-Dev `tools/nip-ca-server/` |
| `NPS-MCP-Ingress` | labacacia | MCP Ingress adapter distribution (`LabAcacia.McpIngress`) | Public | NO — synced from NPS-Dev `compat/mcp-ingress/` |
| `NPS-A2A-Ingress` | labacacia | A2A Ingress adapter distribution (`LabAcacia.A2aIngress`) | Public | NO — synced from NPS-Dev `compat/a2a-ingress/` |
| `NPS-gRPC-Ingress` | labacacia | gRPC Ingress adapter distribution (`LabAcacia.GrpcIngress`) | Public | NO — synced from NPS-Dev `compat/grpc-ingress/` |
| `NPS-Examples` | labacacia | Curated runnable demos (`nwp-graph-walk`, `ingress-playground`, `cross-sdk-interop`) | Public | NO — mirrors NPS-Dev `demos/`; tagged independently |
| `NPS-Cloud-CA` | labacacia | NPS Cloud CA daemon (private — 2027 Q1+ target) | **Private** | NO — synced from NPS-Dev `tools/daemons/nps-cloud-ca/` |
| `NPS-Ledger` | labacacia | K-of-N audit + reputation log daemon | **Private** | NO — synced from NPS-Dev `tools/daemons/nps-ledger/` |
| `nps-orchestrator` | orilynn-studio | Consumer / example orchestrator service | Public | Independent — not synced from NPS-Dev; tracked by `version.yaml` for version parity only |

> **Ingress packages have left the suite train.** The three ingress adapter packages (`LabAcacia.McpIngress`, `LabAcacia.A2aIngress`, `LabAcacia.GrpcIngress`) were **last published at v1.0.0-alpha.16** (2026-07-23). An alpha.17 deprecation release was prepared but never tagged or published, and the packages were removed from the synchronized release train as of **alpha.18** (`expected: skip` in `version.yaml`). Their maintained replacement is the inbound surface of `LabAcacia.NPS.NWP.Bridge`.

---

## Stub Repos (Tracked, Not Yet Released)

These repos exist as public stubs with README placeholder files. CI tracks them in `version.yaml` with `expected: stub` — they are not required to carry a real version but their README must contain a tracking declaration.

| Repo | Organization | Planned purpose |
|------|-------------|-----------------|
| `NPS-Studio` | labacacia | Frame-stream visualizer / debugger (NPS-Dev alpha.6 queue) |
| `NPS-NWP-Manager` | labacacia | Web-based NWM authoring and node management tool — now ships a runnable **v0.1 stub** (`GET /health`, `GET /v1/nodes`) as of alpha.13 |
| `NPS-SDK-CPP` | labacacia | C++ SDK |
| `NPS-SDK-PHP` | labacacia | PHP SDK |

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
| `sync-nps-daemons.sh` | `tools/daemons/{npsd,nps-runner,nps-ingress,nps-registry}/` + `bundle-overlay/` | `labacacia/NPS-Daemons` → Gitee mirror |
| `sync-nip-ca-server.sh` | `tools/nip-ca-server/` | `labacacia/NIP-CA-Server` → Gitee mirror |
| `sync-nps-cloud-ca.sh` | `tools/daemons/nps-cloud-ca/` | `labacacia/NPS-Cloud-CA` (no Gitee) |
| `sync-nps-ledger.sh` | `tools/daemons/nps-ledger/` | `labacacia/NPS-Ledger` (no Gitee) |

SDK sync scripts follow the same pattern but are per-language. The per-adapter ingress sync scripts follow the same pattern too, but are no longer run on the release train — the three ingress repos stopped at their last published release, v1.0.0-alpha.16.

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

*Last reviewed for published packages: v1.0.0-alpha.18*

---

*Last reviewed at suite version: v1.0.0-alpha.18*
