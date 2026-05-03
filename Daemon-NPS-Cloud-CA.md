# Daemon: nps-cloud-ca

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

> **Audience:** NPS Cloud subscribers and operators
> **Distribution note:** `innolotus/nps-cloud-ca` is a **private** repository. This page documents only the protocol-visible interface. Internal product and billing details are in the private repo.
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

`nps-cloud-ca` is the NPS Cloud Certificate Authority — the multi-tenant, billing-aware NID CA operated by INNO LOTUS PTY LTD for NPS Cloud subscribers. It issues NID certificates for agents and nodes across organisations, handles CRL/OCSP, and integrates with the full ACME workflow defined in NPS-RFC-0002.

- **Source:** `NPS-Dev/tools/daemons/nps-cloud-ca/`
- **Distribution:** `innolotus/nps-cloud-ca` — **PRIVATE** (NPS Cloud product)
- **Docker image:** `innolotus/nps-cloud-ca:1.0.0-alpha.5.2` (private registry)
- **Default port:** `:17435` (NIP optional-dedicated per NPS-3 §1)
- **Layer:** L3
- **Timeline:** Ships publicly with NPS Cloud GA, planned 2027 Q1+

---

## nps-cloud-ca vs nip-ca-server

| Aspect | nps-cloud-ca | nip-ca-server |
|--------|--------------|---------------|
| Tenancy | Multi-tenant — each subscriber has isolated NID namespace | Single-tenant — one organisation per deployment |
| Billing | Integrated with NPS Cloud billing | None |
| SLA | Commercial SLA (NPS Cloud agreement) | Best-effort OSS |
| Distribution | Private (`innolotus` org) | Public (`labacacia` org) |
| Availability | NPS Cloud GA (2027 Q1+) | Available now |
| Use for self-hosting today | No — not yet available | Yes — see [Daemon NIP-CA-Server](Daemon-NIP-CA-Server) |

**For any self-hosted CA need today, use [Daemon NIP-CA-Server](Daemon-NIP-CA-Server).**

---

## Implementation status (alpha.5)

The current release is a **Phase 1 deferral skeleton**. The URL surface is present but all issuance endpoints return `NIP-CA-NOT-READY` (HTTP 503) with a pointer to the OSS CA so callers fail informatively. The process name, port, and Docker image tag are stable from alpha.3 to lock in the deployment surface.

The daemon's own X.509 and ACME pipeline is planned for alpha.4 alongside NPS-RFC-0002.

---

## Operator API (protocol-visible interface)

These endpoints define the public protocol surface. Internal product behaviour (billing hooks, tenant isolation, rate limits) is documented in the private repo.

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/v1/nid/issue` | Issue a NID for an agent or node. Requires valid subscriber credentials. Body shape: NIP §5 NID issuance request. |
| `DELETE` | `/v1/nid/{nid}/revoke` | Revoke a NID. Body: `{reason?}`. |
| `GET` | `/v1/nid/{nid}/status` | Query the status of a NID (active / revoked / expired). Returns the current `IdentFrame` metadata. |
| `GET` | `/v1/crl` | Certificate Revocation List for all NIDs issued by this CA instance. |
| `GET` | `/v1/ocsp` | OCSP responder endpoint. |
| `GET` | `/.well-known/nps-ca` | CA discovery document (NID, public key, policy URL). |
| `GET` | `/health` | Standard NPS health envelope. |

---

## `/health` response example

```json
{
  "status": "ok",
  "daemon": "nps-cloud-ca",
  "version": "1.0.0-alpha.5.2",
  "layer": 3,
  "role": "NPS Cloud NID Certificate Authority",
  "port": 17435
}
```

During the Phase 1 skeleton period, the health endpoint returns `200 ok` while issuance endpoints return `503 NIP-CA-NOT-READY`.

---

## Configuration (env vars)

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSCLOUDCA_PORT` | `17435` | TCP port to bind. NIP optional-dedicated per NPS-3 §1. |
| `NPSCLOUDCA_HOST` | `0.0.0.0` | Bind address. |

Full configuration (tenant database, billing integration, CA key management) is documented in the private `innolotus/nps-cloud-ca` repository.

---

## Cross-links

- [Daemon NIP-CA-Server](Daemon-NIP-CA-Server) — the OSS self-hosted CA equivalent; available today
- [Protocol NIP](Protocol-NIP) — Neural Identity Protocol; NID certificate format and issuance semantics
- [Daemon NPS-Ledger](Daemon-NPS-Ledger) — the companion private reputation log daemon

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
