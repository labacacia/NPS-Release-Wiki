# Operator Quickstart: Daemon Bundle

> **Audience:** Operators (devops / SREs deploying NPS infrastructure)
> **Status:** ✅ Latest published bundle — v1.0.0-alpha.13; candidate docs staged for v1.0.0-alpha.14
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

The `nps-daemons` bundle packages the four OSS NPS daemons — **npsd**, **nps-runner**, **nps-ingress**, and **nps-registry** — in a single git repository with a reference `docker-compose.yml`. This is the recommended starting point for operators who want to run a self-hosted NPS cluster. (The private daemons **nps-ledger** and **nps-cloud-ca** ship separately; see [Operator Daemons Reference](Operator-Daemons-Reference).)

**Two install paths:**

| Path | Best for |
|------|---------|
| [Option A: Docker Compose](#option-a-docker-compose) | Isolated deployments, CI, quick evaluation |
| [Option B: Native packages](#option-b-native-packages-systemd--windows-service) | Bare-metal / VM servers, systemd-managed fleets, Windows hosts |

---

## Option A: Docker Compose

### Step 1: Clone the bundle

```bash
git clone https://github.com/labacacia/nps-daemons && cd nps-daemons
```

The repository root contains:

| Path | Description |
|------|-------------|
| `docker-compose.yml` | Reference four-service composition |
| `deploy/docker-compose/` | Curated Compose overlays (dev / prod, optional nip-ca-server sidecar) |
| `deploy/systemd/` | systemd unit files for native installs |
| `Makefile` | Convenience targets — `make up` / `make down` / `make install-systemd` |
| `npsd/` | npsd daemon source and Dockerfile |
| `nps-runner/` | nps-runner daemon source and Dockerfile |
| `nps-ingress/` | nps-ingress daemon source and Dockerfile |
| `nps-registry/` | nps-registry daemon source and Dockerfile |
| `CHANGELOG.md` | Per-release notes |

> **Makefile shortcuts (alpha.6+).** From the repository root: `make up` brings the
> Compose stack up (wraps `docker compose up -d`), `make down` tears it down, and
> `make install-systemd` installs the units from `deploy/systemd/` for a native
> deployment. See `deploy/docker-compose/` and `deploy/systemd/` for the underlying
> files.

---

## Step 2: Review `docker-compose.yml`

The compose file defines one service per daemon. The key port bindings are:

| Service | Internal port | External default | Notes |
|---------|--------------|-----------------|-------|
| `npsd` | `17433` | `127.0.0.1:17433` | Loopback-only by default — do not expose directly |
| `nps-ingress` | `8080` | `${NPS_INGRESS_PORT:-8080}` | Internet-facing HTTP ingress |
| `nps-registry` | `17436` | `${NPS_REGISTRY_PORT:-17436}` | NDP cross-machine discovery |
| `nps-runner` | — | (none) | Connects outbound to npsd; no inbound port |

> **Production note.** Place `nps-ingress` behind nginx, Caddy, or Traefik for TLS termination. Run `npsd` on every machine that hosts NPS workers. Run `nps-registry` once per cluster behind an internal load balancer.

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

#### nps-ingress

| Variable | Default | Purpose |
|----------|---------|---------|
| `NPSINGRESS_HOST` | `0.0.0.0` | Bind address. The ingress daemon is intentionally Internet-facing. |
| `NPSINGRESS_PORT` | `8080` | TCP port. Production deployments terminate TLS at `:443` via a reverse proxy. |

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

# nps-ingress — HTTP ingress
curl -s http://localhost:8080/health | jq

# nps-registry — NDP discovery registry
curl -s http://localhost:17436/health | jq
```

Expected npsd response shape:

```json
{
  "status": "ok",
  "daemon": "npsd",
  "version": "1.0.0-alpha.13",
  "layer": "L1",
  "role": "node",
  "port": 17433,
  "host_nid": "urn:nps:host:...",
  "uptime_s": 42
}
```

### Kubernetes-style probes and metrics

As of alpha.6+, **npsd** and **nip-ca-server** also expose dedicated operational
endpoints alongside `/health`:

| Endpoint | Purpose |
|----------|---------|
| `GET /healthz` | Liveness probe (process is up) |
| `GET /readyz` | Readiness probe (dependencies ready, accepting traffic) |
| `GET /metrics` | Prometheus-format metrics (frame counters, inbox depth, handshake latency, etc.) |

```bash
curl -s http://localhost:17433/healthz
curl -s http://localhost:17433/readyz
curl -s http://localhost:17433/metrics
```

> **nip-ca-server `/metrics` port note.** From alpha.6+, `nip-ca-server` serves
> `/metrics` on its **management port `17436`**, not the public CA port `17435`.
> The public CA port no longer exposes `/metrics`. Scrape
> `http://<ca-host>:17436/metrics` and keep `17436` off the public Internet.

### Graceful shutdown

On `SIGTERM`, npsd and nip-ca-server perform a **graceful drain** (default 30 s):
they stop accepting new connections, finish in-flight requests, flush inbox/state,
then exit. Configure your orchestrator's termination grace period to ≥ 30 s.

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
   image: labacacia/npsd:1.0.0-alpha.13        # change to target version
   image: labacacia/nps-runner:1.0.0-alpha.13
   image: labacacia/nps-ingress:1.0.0-alpha.13
   image: labacacia/nps-registry:1.0.0-alpha.13
   ```

2. Back up all named volumes (see above).

3. Pull new images and restart:

   ```bash
   docker compose pull && docker compose up -d
   ```

4. Confirm health on all ports.

---

## Common bring-up errors (Docker)

| Symptom | Likely cause | Resolution |
|---------|-------------|-----------|
| `bind: address already in use` on port 17433, 17436, or 8080 | Another process is already using that port | Change `NPSD_PORT` / `NPSREGISTRY_PORT` / `NPSINGRESS_PORT`, or stop the conflicting process |
| `nps-ingress` starts but returns `502` upstream errors | `npsd` is not yet ready or not reachable at `127.0.0.1:17433` | Check `depends_on` ordering; confirm `npsd` health endpoint responds |
| NDP `Resolve` queries time out | Firewall blocking UDP or the NDP registry port | Ensure port 17436 TCP is open between cluster machines; UDP is used by NDP for multicast but the registry daemon uses TCP |
| `npsd` exits immediately on first start | `NPSD_DATA_DIR` is not writable, or the keypair file has wrong permissions | Verify the volume mount; the root key file must be `0600` (TC-N1-NIP-01) |
| nps-runner workers not spawning | `NPSD_URL` points at the wrong address inside the container | Use the Docker service name: `http://npsd:17433` (not `localhost`) |

---

## Option B: Native packages (systemd / Windows service)

Native packages are self-contained binaries — no Docker, no .NET runtime installation required. Each package registers the daemon as a system service that starts on boot.

Download from the [nps-daemons releases page](https://github.com/labacacia/nps-daemons/releases).

### Ubuntu / Debian (amd64)

```bash
# Set the suite version (Debian format: ~ separates pre-release)
DEB_VER="1.0.0~alpha.13"
SUITE_VER="1.0.0-alpha.13"

for pkg in npsd nps-runner nps-ingress nps-registry; do
    curl -LO "https://github.com/labacacia/nps-daemons/releases/download/v${SUITE_VER}/${pkg}_${DEB_VER}_amd64.deb"
    sudo dpkg -i "${pkg}_${DEB_VER}_amd64.deb"
done
```

The installer:
- Creates a dedicated system user/group per daemon (`npsd`, `npsrunner`, `npsgw`, `npsreg`)
- Installs the binary to `/opt/labacacia/<daemon>/`
- Installs the systemd unit to `/lib/systemd/system/<daemon>.service`
- Creates `/var/lib/nps/<daemon>/` (owned by the service user, mode 750)
- Enables and starts the service automatically

**Configuration** (environment overrides, preserved on upgrade):

```bash
# Edit the env file for any daemon, then restart:
sudo systemctl edit --force npsd.service
# — or —
sudo nano /etc/nps/npsd/env
sudo systemctl restart npsd
```

Example `/etc/nps/npsd/env`:

```bash
# Uncomment to override defaults
#ASPNETCORE_URLS=http://127.0.0.1:17433
#NPSD_DATA_DIR=/var/lib/nps/npsd
```

**Verify:**

```bash
sudo systemctl status npsd nps-runner nps-ingress nps-registry
curl -s http://localhost:17433/health | jq
```

**Uninstall:**

```bash
sudo apt remove npsd nps-runner nps-ingress nps-registry
```

Data directories under `/var/lib/nps/` are not removed on uninstall (`apt purge` removes them).

---

### Fedora / RHEL (x86_64)

```bash
SUITE_VER="1.0.0-alpha.13"
RPM_VER="1.0.0"
RPM_REL="0.alpha.13"   # for stable releases: "1"

for pkg in npsd nps-runner nps-ingress nps-registry; do
    curl -LO "https://github.com/labacacia/nps-daemons/releases/download/v${SUITE_VER}/${pkg}-${RPM_VER}-${RPM_REL}.x86_64.rpm"
    sudo rpm -i "${pkg}-${RPM_VER}-${RPM_REL}.x86_64.rpm"
done
```

systemd units install to `/usr/lib/systemd/system/`. Config and data directories are the same as Debian.

**Verify:**

```bash
sudo systemctl status npsd
curl -s http://localhost:17433/health | jq
```

**Uninstall:**

```bash
sudo rpm -e npsd nps-runner nps-ingress nps-registry
```

---

### Windows (x64, MSI)

Each daemon ships as a per-daemon `.msi` installer. Run as Administrator.

```powershell
$ver = "1.0.0-alpha.13"

foreach ($pkg in @("npsd","nps-runner","nps-ingress","nps-registry")) {
    $file = "$pkg-$ver-win-x64.msi"
    Invoke-WebRequest `
        -Uri "https://github.com/labacacia/nps-daemons/releases/download/v$ver/$file" `
        -OutFile $file
    Start-Process msiexec.exe -ArgumentList "/i `"$file`" /quiet /norestart" -Wait
}

# Services start automatically; verify:
Get-Service npsd, nps-runner, nps-ingress, nps-registry
```

The MSI:
- Installs to `%ProgramFiles%\LabAcacia\<daemon>\`
- Creates `%ProgramData%\LabAcacia\<daemon>\` as the service data directory
- Registers the service under `NT SERVICE\<daemon>` (virtual account — no password needed)
- Configures auto-restart on failure (3 attempts, 10 s delay)

**Configuration** (Windows environment variables):

Open `services.msc`, right-click the daemon → Properties → Log On tab to configure an alternate account, or set environment variables via the registry:

```powershell
# Set an env var for npsd (takes effect after service restart):
[Environment]::SetEnvironmentVariable(
    "NPSD_DATA_DIR", "D:\nps\npsd",
    [System.EnvironmentVariableTarget]::Machine
)
Restart-Service npsd
```

**Uninstall:**

```powershell
foreach ($pkg in @("npsd","nps-runner","nps-ingress","nps-registry")) {
    Get-Package $pkg -ErrorAction SilentlyContinue | Uninstall-Package -Force
}
```

---

### Common bring-up errors (native packages)

| Symptom | Likely cause | Resolution |
|---------|-------------|-----------|
| Service fails to start (Linux) | `/var/lib/nps/<daemon>/` missing or wrong permissions | Check `systemctl status <daemon>` and `journalctl -u <daemon> -n 50`; run `sudo chown <user>:<group> /var/lib/nps/<daemon>` |
| Service fails to start (Windows) | Data directory not writable by `NT SERVICE\<daemon>` | Verify `%ProgramData%\LabAcacia\<daemon>` exists with the service account having full control |
| Port already in use | Another process using port 17433 / 8080 / 17436 | Override via `/etc/nps/<daemon>/env` (Linux) or `SetEnvironmentVariable` (Windows) |
| `dpkg-i` errors about dependency | `libc6` or `libstdc++` version mismatch | Packages are fully self-contained; run `sudo apt install -f` to fix missing system deps |

---

## See also

- [Operator Daemons Reference](Operator-Daemons-Reference) — per-daemon env vars, API surfaces, and scaling notes
- [Operator AaaS Profile](Operator-AaaS-Profile) — how to expose NPS-compatible endpoints to AI agents

---

*Last reviewed for published packages: v1.0.0-alpha.13; candidate delta staged: v1.0.0-alpha.14*
