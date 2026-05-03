# Operator Quickstart: Daemon Bundle

> **Audience:** Operators (devops / SREs deploying NPS infrastructure)
> **Status:** ✅ Content complete — v1.0.0-alpha.5.2
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

The `nps-daemons` bundle packages the four OSS NPS daemons — **npsd**, **nps-runner**, **nps-gateway**, and **nps-registry** — in a single git repository with a reference `docker-compose.yml`. This is the recommended starting point for operators who want to run a self-hosted NPS cluster. (The private daemons **nps-ledger** and **nps-cloud-ca** ship separately; see [Operator Daemons Reference](Operator-Daemons-Reference).)

---

## Step 1: Clone the bundle

```bash
git clone https://github.com/labacacia/nps-daemons && cd nps-daemons
```

The repository root contains:

| Path | Description |
|------|-------------|
| `docker-compose.yml` | Reference four-service composition |
| `npsd/` | npsd daemon source and Dockerfile |
| `nps-runner/` | nps-runner daemon source and Dockerfile |
| `nps-gateway/` | nps-gateway daemon source and Dockerfile |
| `nps-registry/` | nps-registry daemon source and Dockerfile |
| `CHANGELOG.md` | Per-release notes |

---

## Step 2: Review `docker-compose.yml`

The compose file defines one service per daemon. The key port bindings are:

| Service | Internal port | External default | Notes |
|---------|--------------|-----------------|-------|
| `npsd` | `17433` | `127.0.0.1:17433` | Loopback-only by default — do not expose directly |
| `nps-gateway` | `8080` | `${NPS_GATEWAY_PORT:-8080}` | Internet-facing HTTP ingress |
| `nps-registry` | `17436` | `${NPS_REGISTRY_PORT:-17436}` | NDP cross-machine discovery |
| `nps-runner` | — | (none) | Connects outbound to npsd; no inbound port |

> **Production note.** Place `nps-gateway` behind nginx, Caddy, or Traefik for TLS termination. Run `npsd` on every machine that hosts NPS workers. Run `nps-registry` once per cluster behind an internal load balancer.

### Required environment variables

Configure these before `docker compose up`. Set them in a `.env` file at the repository root or via your secrets management system. Never hardcode values in `docker-compose.yml`.

#### npsd

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSD_HOST` | `127.0.0.1` | Bind address. Use `0.0.0.0` only inside an isolated network namespace. |
| `NPSD_PORT` | `17433` | TCP port to bind. |
| `NPSD_DATA_DIR` | `~/.local/share/npsd` | Persistent state: root Ed25519 keypair + sub-NID SQLite database. |
| `NPSD_HOST_NID_PREFIX` | `urn:nps:host:{HostFingerprint}` | NID prefix for minting sub-NIDs. Override only if the host is registered under a different NID with an upstream CA. |
| `NPSD_SUB_NID_VALIDITY_DAYS` | `7` | Default validity window for issued sub-NIDs. |
| `NPSD_MAX_INBOX_DEPTH_PER_NID` | `1024` | Max pending messages per NID before deposits return `429`. |
| `NPSD_MAX_INBOX_MESSAGE_BYTES` | `65536` | Per-message payload cap (matches NCP default frame size). |
| `NPSD_MAX_INBOX_WAIT_SECONDS` | `30` | Maximum long-poll wait time. Larger values are clamped. |

#### nps-runner

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSD_URL` | `http://127.0.0.1:17433` | URL of the npsd instance this runner connects to. |
| `NPS_RUNNER_AGENT_ID` | `nps-runner` | Identifier used when self-registering the runner's sub-NID. |
| `NPS_RUNNER_MAX_CONCURRENT_WORKERS` | `8` | Cap on simultaneously running worker processes. |
| `NPS_RUNNER_LOG_DIR` | `/tmp/nps-runner-logs` | Directory for per-worker `{task_id}.log` output files. |

#### nps-gateway

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSGATEWAY_HOST` | `0.0.0.0` | Bind address. The gateway is intentionally Internet-facing. |
| `NPSGATEWAY_PORT` | `8080` | TCP port. Production deployments terminate TLS at `:443` via a reverse proxy. |

#### nps-registry

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSREGISTRY_HOST` | `0.0.0.0` | Bind address. |
| `NPSREGISTRY_PORT` | `17436` | TCP port. |
| `NPSREGISTRY_SQLITE_PATH` | *(in-memory)* | Path to the SQLite file for persistent storage. Omit for ephemeral dev use. |

### Optional environment variables

These apply when you add **nps-ledger** (private) to your cluster and want it to participate in the STH gossip federation:

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSLEDGER_PEERS` | *(empty)* | Comma-separated `host:port` list of peer ledger operators to exchange STH gossip with. Example: `log2.example.com:17440,log3.example.com:17440`. |
| `NPSLEDGER_GOSSIP_INTERVAL_S` | `30` | Background gossip push-pull cycle interval in seconds. Minimum 10, maximum 3600. |

---

## Step 3: Start the bundle

```bash
docker compose up -d
```

To start a single daemon:

```bash
docker compose up -d npsd
```

To tail logs:

```bash
docker compose logs -f nps-runner
```

---

## Health checks

Verify the cluster is up:

```bash
# npsd — NPS Daemon (Layer 1)
curl -s http://localhost:17433/health | jq

# nps-gateway — HTTP ingress
curl -s http://localhost:8080/health | jq

# nps-registry — NDP discovery registry
curl -s http://localhost:17436/health | jq
```

Expected npsd response shape:

```json
{
  "status": "ok",
  "daemon": "npsd",
  "version": "1.0.0-alpha.5.2",
  "layer": "L1",
  "role": "node",
  "port": 17433,
  "host_nid": "urn:nps:host:...",
  "uptime_s": 42
}
```

---

## Data persistence

The compose file declares a named Docker volume `npsd-data` mounted at `/data` inside the `npsd` container. This volume holds:

- `root.ed25519.pkcs8` — the host's root Ed25519 keypair (POSIX mode `0600`)
- `sub-nids.sqlite` — all issued sub-NID records

**Before upgrading**, back up the volume by copying its contents:

```bash
# Find the volume's mount point on the host
docker volume inspect nps-daemons_npsd-data --format '{{ .Mountpoint }}'

# Copy (run as root / sudo if needed)
cp -a /var/lib/docker/volumes/nps-daemons_npsd-data/_data /backup/npsd-data-$(date +%Y%m%d)
```

`nps-registry` and `nps-ledger` (if deployed) each have their own volumes. Back up similarly before any upgrade.

---

## Upgrade procedure

1. Pin all services to the new suite version in `docker-compose.yml`:

   ```yaml
   image: labacacia/npsd:1.0.0-alpha.5.2        # change to target version
   image: labacacia/nps-runner:1.0.0-alpha.5.2
   image: labacacia/nps-gateway:1.0.0-alpha.5.2
   image: labacacia/nps-registry:1.0.0-alpha.5.2
   ```

2. Back up all named volumes (see above).

3. Pull new images and restart:

   ```bash
   docker compose pull && docker compose up -d
   ```

4. Confirm health on all ports.

---

## Common bring-up errors

| Symptom | Likely cause | Resolution |
|---------|-------------|-----------|
| `bind: address already in use` on port 17433, 17436, or 8080 | Another process is already using that port | Change `NPSD_PORT` / `NPSREGISTRY_PORT` / `NPSGATEWAY_PORT`, or stop the conflicting process |
| `nps-gateway` starts but returns `502` upstream errors | `npsd` is not yet ready or not reachable at `127.0.0.1:17433` | Check `depends_on` ordering; confirm `npsd` health endpoint responds |
| NDP `Resolve` queries time out | Firewall blocking UDP or the NDP registry port | Ensure port 17436 TCP is open between cluster machines; UDP is used by NDP for multicast but the registry daemon uses TCP |
| `npsd` exits immediately on first start | `NPSD_DATA_DIR` is not writable, or the keypair file has wrong permissions | Verify the volume mount; the root key file must be `0600` (TC-N1-NIP-01) |
| nps-runner workers not spawning | `NPSD_URL` points at the wrong address inside the container | Use the Docker service name: `http://npsd:17433` (not `localhost`) |

---

## See also

- [Operator Daemons Reference](Operator-Daemons-Reference) — per-daemon env vars, API surfaces, and scaling notes
- [Operator AaaS Profile](Operator-AaaS-Profile) — how to expose NPS-compatible endpoints to AI agents

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
