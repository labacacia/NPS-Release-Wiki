# Daemon: nps-gateway

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

> **Audience:** Operators
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

`nps-gateway` is the public Internet ingress daemon for the NPS suite. It accepts NCP-over-HTTP (and, in production, NCP-over-TLS) connections from external clients, terminates TLS, performs rate limiting and authentication, and forwards frames upstream to [Daemon NPSd](Daemon-NPSd) at port 17433. Unlike `npsd` — which binds loopback only — `nps-gateway` is intentionally Internet-facing.

- **Source:** `NPS-Dev/tools/daemons/nps-gateway/`
- **Distribution:** `labacacia/nps-daemons` (public), assembled via `tools/release/sync-nps-daemons.sh`
- **Docker image:** `labacacia/nps-gateway:1.0.0-alpha.5.2`
- **Default port:** `:8080` (HTTP). Production deployments terminate TLS on `:443` via a reverse proxy (nginx, Caddy, or Traefik) in front of this daemon.
- **Layer:** L2

---

## NCP HTTP mode

In HTTP mode, each `POST` request body carries exactly one NPS frame; the response body carries exactly one frame. This makes NPS deployable over any HTTP/1.1 or HTTP/2 transport — useful for firewall-restricted environments that cannot establish raw TCP connections to port 17433.

Path-based routing: `nps-gateway` examines the `Host` header and request path to determine which backing `npsd` node to forward to. In a single-node deployment all traffic routes to `http://127.0.0.1:17433`. Multi-node routing is configured via the upstream registry at `nps-registry`.

---

## Implementation status (alpha.5)

The current release ships a **Phase 1 skeleton**: a public HTTP listener with a `/health` endpoint and stable deployment surface (process name, Docker image tag, port). This skeleton has been present since alpha.3 to keep the deployment topology stable from the beginning of the daemon ecosystem.

Full ingress logic — TLS termination, rate limiting, NeuronHub-customer authentication, CGN debit triggering, NPS-RFC-0004 reputation checks, and Anchor Node middleware wiring per NPS-CR-0001 — is planned for alpha.5 and later milestones. The `nps-gateway` process MAY host an Anchor Node middleware via `NPS.NWP.Anchor`; that wiring is deferred until the Anchor Node middleware is stable.

---

## Naming note

This is the **process** called `nps-gateway`. The spec-level role of "cluster control plane that routes NPS frames into a NOP DAG" has been renamed **Anchor Node** in the NWP specification by [NPS-CR-0001](https://github.com/labacacia/NPS-Release/blob/main/spec/cr/NPS-CR-0001-anchor-bridge-split.md). The two concepts are related but distinct: `nps-gateway` is the deployment binary; Anchor Node is the spec-level role it hosts.

---

## TLS termination

The `nps-gateway` container itself speaks plain HTTP on port 8080. Production deployments place a TLS-terminating reverse proxy in front:

```
Internet → nginx/Caddy/Traefik :443 (TLS) → nps-gateway :8080 (HTTP) → npsd :17433
```

Configure your reverse proxy to set `X-Forwarded-For` and `X-Forwarded-Proto` so that `nps-gateway` can log the real client address and detect the original scheme.

---

## `/health` response example

```json
{
  "status": "ok",
  "daemon": "nps-gateway",
  "version": "1.0.0-alpha.5.2",
  "layer": 2,
  "role": "internet-ingress",
  "port": 8080
}
```

---

## Configuration (env vars)

| Variable | Default | Required | Purpose |
|----------|---------|----------|---------|
| `NPSGATEWAY_PORT` | `8080` | no | TCP port to bind. Production deployments terminate TLS externally; this daemon only listens HTTP. |
| `NPSGATEWAY_HOST` | `0.0.0.0` | no | Bind address. The default is public — unlike `npsd`, `nps-gateway` is intentionally Internet-facing. |

### docker-compose.yml service definition (bundle-overlay)

```yaml
nps-gateway:
  image: labacacia/nps-gateway:1.0.0-alpha.5.2
  restart: unless-stopped
  ports:
    - "${NPS_GATEWAY_PORT:-8080}:8080"
  depends_on:
    - npsd
```

The host port is configurable via the `NPS_GATEWAY_PORT` environment variable at compose launch time (default `8080`).

---

## HA and load-balancer placement

`nps-gateway` is stateless (all state lives in `npsd` and `nps-registry`). It is safe to run multiple instances behind a load balancer — no session affinity is required. Each instance independently routes to the same backing `npsd`.

Recommended production topology:

```
LB :443 (TLS) ─┬─ nps-gateway-1 :8080 ─┐
               └─ nps-gateway-2 :8080 ─┴─ npsd :17433
```

---

## Common operational issues

**Port 8080 already in use**

Set `NPSGATEWAY_PORT` to an available port, or stop the conflicting service. Check with `ss -tlnp | grep 8080`.

**All requests return 502 or hang**

`npsd` at `127.0.0.1:17433` is not reachable from inside the `nps-gateway` container. In Docker, ensure both containers share a network (in docker-compose, services in the same file share a default bridge) and that `npsd` is healthy before `nps-gateway` starts handling traffic.

---

## Cross-links

- [Protocol NCP](Protocol-NCP) — wire format; HTTP mode is defined in NCP §2.2
- [Daemon NPSd](Daemon-NPSd) — upstream target for forwarded frames
- [Daemon NPS-Registry](Daemon-NPS-Registry) — topology store queried for multi-node routing
- [Operator Daemons Reference](Operator-Daemons-Reference)
- [Operator Quickstart Bundle](Operator-Quickstart-Bundle)

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
