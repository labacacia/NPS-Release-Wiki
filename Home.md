# NPS — Neural Protocol Suite Wiki

> **Status:** ✅ Wiki aligned for the v1.0.0-alpha.14 release.

> 🌐 New to NPS? Start at the [overview site](https://nps.labacacia.com) for a 5-minute orientation, then come back here for deep-dives.

The **Neural Protocol Suite (NPS)** is a 5-protocol family for Agent-to-Agent
and Agent-to-Service communication. The canonical specification lives in
[`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release); this
wiki is the *narrative* layer — tutorials, how-tos, deployment guides, and
cross-cutting references that don't fit naturally inside a normative spec
document.

## Choose your starting point

| If you are… | Start here |
|---|---|
| **New to NPS** and want the 5-minute orientation | [What Is NPS](What-Is-NPS) → [Protocol Stack Architecture](Protocol-Stack-Architecture) → [Glossary](Glossary) |
| A **developer** building an Agent or Node against NPS | [SDK Quickstart](SDK-Quickstart) → pick your language ([.NET](SDK-DotNet) · [Python](SDK-Python) · [TypeScript](SDK-TypeScript) · [Java](SDK-Java) · [Go](SDK-Go) · [Rust](SDK-Rust)) → walk [Building an Anchor Node](SDK-Building-an-Anchor-Node) |
| An **operator** deploying NPS daemons or running an AaaS service | [Operator Quickstart Bundle](Operator-Quickstart-Bundle) → [Daemons Reference](Operator-Daemons-Reference) → individual daemon pages → [AaaS Profile](Operator-AaaS-Profile) |
| Looking to **see code in action** | [Example: ingress-playground](Example-Ingress-Playground) · [Example: cross-sdk-interop](Example-Cross-SDK-Interop) · [Example: nwp-graph-walk](Example-NWP-Graph-Walk) |
| A **protocol designer** writing an RFC or CR against the spec | [Specs Index](Specs-Index) → [RFC Process](RFC-Process) → [CR Process](CR-Process) |
| A **contributor** opening a PR or shepherding a release | [Contributing Guide](Contributing-Guide) → [Release Process](Release-Process) → [Repository Topology](Repository-Topology) |

## Reference (cross-cutting)

- [Reference: Error Codes](Reference-Error-Codes) — every `<DOMAIN>-<NAME>` code, mapped to status code + HTTP
- [Reference: Status Codes](Reference-Status-Codes) — `NPS-*` status family
- [Reference: Frame Registry](Reference-Frame-Registry) — every frame type byte and its owning protocol
- [Reference: Cognon Budget](Reference-Cognon-Budget) — token-cost model (formerly NPT, renamed in alpha.5.2)

## Per-protocol pages

- [NCP](Protocol-NCP) — Neural Connection Protocol (transport, native + HTTP modes)
- [NWP](Protocol-NWP) — Neural Web Protocol (HTTP-equivalent surface; topology queries)
- [NIP](Protocol-NIP) — Neural Identity Protocol (NID certs, assurance levels, reputation log)
- [NDP](Protocol-NDP) — Neural Discovery Protocol (DNS TXT, AnnounceFrame, GraphFrame)
- [NOP](Protocol-NOP) — Neural Orchestration Protocol (DAG dispatch, delegation chains)

## Per-language SDKs

- [.NET / C#](SDK-DotNet)
- [Python](SDK-Python)
- [TypeScript](SDK-TypeScript)
- [Java](SDK-Java)
- [Go](SDK-Go)
- [Rust](SDK-Rust)

## Per-daemon pages

- [npsd](Daemon-NPSd) — orchestration runtime
- [nps-runner](Daemon-NPS-Runner) — task executor
- [nps-ingress](Daemon-NPS-Ingress) — HTTP-mode ingress
- [nps-registry](Daemon-NPS-Registry) — node / member registry
- [nps-ledger](Daemon-NPS-Ledger) — NID reputation log + STH gossip
- [nps-cloud-ca](Daemon-NPS-Cloud-CA) — private NPS Cloud CA
- [nip-ca-server](Daemon-NIP-CA-Server) — OSS reference NIP CA
- [bundle-overlay](Daemon-Bundle-Overlay) — release packaging meta

## Suite version

This wiki tracks the suite version declared in
[`NPS-Release/version.yaml`](https://github.com/labacacia/NPS-Release/blob/main/version.yaml)
— latest published packages are **v1.0.0-alpha.14** (released 2026-06-26).
The wiki covers the **v1.0.0-alpha.14 release** documentation
boundary: typed remote NIP CA clients, native-mode NWP serving helpers,
conformance manifests, live revocation hooks, native NCP TLS/mTLS hardening,
signed CRL output, and transport-neutral observability. Pages should be
re-reviewed at each suite release; see the `## TODO` block on each page for
the version-alignment checklist.

*Wiki last updated: v1.0.0-alpha.14 release docs (2026-06-26)*

---

> **Wiki hygiene rule.** This wiki complements but never substitutes for the
> normative `spec/` documents. When a wiki page and a `spec/` document
> disagree, **the spec wins**; open an issue against the wiki page to fix it.
