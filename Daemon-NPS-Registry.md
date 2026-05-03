# Daemon: nps-registry

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

> **Audience:** Operators
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

`nps-registry` is the cross-machine NDP discovery registry for an NPS cluster. Where `npsd` knows only about its own host-local sessions, `nps-registry` aggregates AnnounceFrame records from multiple machines and answers NDP `Resolve` and `Graph` queries cluster-wide. It is the topology store that Anchor Node middleware queries to serve NWP `topology.snapshot` and `topology.stream` requests, and it is required for AaaS L2 conformance requirement L2-08.

- **Source:** `NPS-Dev/tools/daemons/nps-registry/`
- **Distribution:** `labacacia/nps-daemons` (public), assembled via `tools/release/sync-nps-daemons.sh`
- **Docker image:** `labacacia/nps-registry:1.0.0-alpha.5.2`
- **Default port:** `:17436` (NDP optional-dedicated port per NPS-4)
- **Layer:** L2

---

## What nps-registry stores

Each record corresponds to one NDP `AnnounceBody` deposited via `POST /v1/announce`. Records are indexed by NID and capability. A monotonic per-cluster `seq` counter increments on every announce or TTL eviction, so clients can detect changes cheaply by polling `GET /v1/graph` and comparing `seq`.

TTL-based lazy expiry: records are not deleted by a background timer. Instead, expired records are filtered out on read. Each announce refreshes the TTL; the default TTL is the value in the announce body, or 300 seconds if unset.

---

## API surface

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/v1/announce` | Register or refresh a node announcement. Accepts an NDP `AnnounceBody`; returns the stored entry. TTL defaults to the announced value or 300 s if unset. |
| `GET` | `/v1/resolve?nid=<nid>` | Resolve a single NID to its current announcement. Returns `404` if unknown or expired. |
| `GET` | `/v1/graph` | Return all live (non-expired) announcements as a JSON array, plus a `seq` monotonic counter for client-side change detection. |
| `GET` | `/health` | Liveness probe. Returns `status`, `storage`, and current graph `seq`. |

---

## `/health` response example

```json
{
  "status": "ok",
  "daemon": "nps-registry",
  "version": "1.0.0-alpha.5.2",
  "layer": 2,
  "role": "NDP cross-machine discovery registry",
  "storage": "sqlite",
  "seq": 42
}
```

---

## Configuration (env vars)

| Variable | Default | Required | Purpose |
|----------|---------|----------|---------|
| `NPSREGISTRY_PORT` | `17436` | no | TCP port to bind. NDP optional-dedicated per NPS-4. |
| `NPSREGISTRY_HOST` | `0.0.0.0` | no | Bind address. A registry is intentionally network-facing so all machines in the cluster can reach it. |
| `NPSREGISTRY_SQLITE_PATH` | *(in-memory)* | no | Path to the SQLite database file. When unset, an ephemeral in-memory store is used (useful for dev/test; data is lost on restart). Set to a persistent path for production. |

### docker-compose.yml service definition (bundle-overlay)

```yaml
nps-registry:
  image: labacacia/nps-registry:1.0.0-alpha.5.2
  restart: unless-stopped
  ports:
    - "${NPS_REGISTRY_PORT:-17436}:17436"
```

To enable persistence, add a volume mount and set `NPSREGISTRY_SQLITE_PATH`:

```yaml
nps-registry:
  image: labacacia/nps-registry:1.0.0-alpha.5.2
  restart: unless-stopped
  ports:
    - "17436:17436"
  volumes:
    - registry-data:/data
  environment:
    NPSREGISTRY_SQLITE_PATH: /data/registry.db
```

---

## Member-registry semantics

Nodes `ANNOUNCE` to the registry by posting an NDP `AnnounceBody`. The registry:

1. Validates the presence of a signature on the announce body (structural check in alpha.5; full Ed25519 verification against the node's IdentFrame is a follow-up milestone).
2. Upserts the record keyed by NID, resetting the TTL on each call.
3. Bumps the cluster graph `seq` counter.

When a node goes offline, its record expires after its TTL elapses. Nodes are expected to re-announce before expiry (typically every TTL/2 seconds).

---

## Relationship to NWP topology queries (NPS-CR-0002)

NWP topology queries (`topology.snapshot`, `topology.stream`) from Anchor Nodes are backed by `nps-registry`. The Anchor Node middleware issues `GET /v1/graph` or `GET /v1/resolve?nid=…` to retrieve member data and returns it to the requesting agent.

Sub-Anchor recursion (NPS-CR-0002 OQ-1) allows an Anchor Node to fan out topology queries to sub-Anchor registries. This is optional at L2 and not required for L1 conformance.

---

## Scaling

`nps-registry` is typically run **once per cluster**, fronted by an internal load balancer for HA. It is network-facing by default (`0.0.0.0`) and should be placed on an internal network segment not reachable from the public Internet.

For single-machine deployments, you can omit `nps-registry` entirely — `npsd`'s local session data is sufficient and the registry defaults to the in-memory ephemeral store if no path is set.

L2 cross-machine federation (HA cluster mode / gossip between registry instances) is planned for a future milestone.

---

## Common operational issues

**Resolve returns 404 for a NID that should be registered**

Either the node's announcement TTL has expired, or the node has not called `POST /v1/announce` for this registry instance. Check with `GET /v1/graph` to see all live records. Ensure nodes are re-announcing before their TTL lapses.

**`seq` counter not advancing after announcements**

The registry may be using an in-memory store and has been restarted (clearing all state), or announcements are being rejected. Check the response body from `POST /v1/announce` for an error payload.

---

## Cross-links

- [Protocol NDP](Protocol-NDP) — NPS Discovery Protocol; the announce/resolve/graph semantics
- [Operator AaaS Profile](Operator-AaaS-Profile) — L2-08 conformance requirement backed by this daemon
- [Daemon NPS-Gateway](Daemon-NPS-Gateway) — Anchor Node ingress that queries this registry for routing
- [Operator Daemons Reference](Operator-Daemons-Reference)
- [Operator Quickstart Bundle](Operator-Quickstart-Bundle)

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
