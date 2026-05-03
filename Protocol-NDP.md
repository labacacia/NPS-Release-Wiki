# Protocol: NDP — Neural Discovery Protocol

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

**Spec**: `spec/NPS-4-NDP.md` v0.6 · **Port**: 17433 (shared) / 17436 (optional dedicated)

NDP is DNS for the AI era. Where DNS maps human-readable domain names to IP addresses, NDP maps NPS identities to physical endpoints and capability profiles — without a central registry. Nodes announce their own presence and capabilities; resolvers cache those announcements with a TTL. Agents discover nodes by querying the local registry, DNS TXT records, or the NPS Cloud Registry, in that priority order.

Related: [Protocol NWP](Protocol-NWP) | [SDK Building a Bridge Node](SDK-Building-a-Bridge-Node) | [Operator Daemons Reference](Operator-Daemons-Reference)

---

## Design Philosophy

NDP is explicitly decentralized: there is no required central registry. Any node can announce itself and any resolver can cache the result. The three resolution modes provide a fallback chain:

| Mode | Use case | Priority |
|------|----------|----------|
| Local registry (UDP multicast) | Intranet / LAN discovery | Highest |
| DNS TXT records | Public internet, distributed, no infrastructure needed | Medium |
| NPS Cloud Registry | Global centralized fallback | Lowest |

This layered approach lets an intranet deployment work entirely without external dependencies, a public deployment work without any custom registry infrastructure, and large production deployments use the NPS Cloud Registry for global reach.

---

## DNS TXT Discovery (alpha.5)

A Node MAY publish discovery information in standard DNS TXT records, enabling zero-infrastructure discovery for any Agent that can perform a DNS lookup.

**Node discovery record format:**
```
_nps-node.api.example.com.  IN TXT  "v=nps1 type=memory port=17434 nid=urn:nps:node:api.example.com:products fp=sha256:a3f9..."
```

**CA discovery record format:**
```
_nps-ca.mycompany.com.      IN TXT  "v=nps1 ca=https://ca.mycompany.com/.well-known/nps-ca"
```

**TXT record keys:**

| Key | Required | Description |
|-----|----------|-------------|
| `v` | yes | Version; fixed `nps1` (compact DNS convention; distinct from the NCP preamble `NPS/1.0\n`) |
| `nid` | yes | Node NID |
| `type` | no | Node type: `memory` / `action` / `complex` |
| `port` | no | Port (default 17434) |
| `fp` | no | Node certificate fingerprint for pinning |
| `ca` | conditional | CA discovery endpoint (required for CA records) |

**SDK resolver functions** (`resolveViaDns` / `ResolveViaDns` / `resolve_via_dns`) were added across all six SDKs in alpha.5. Each SDK supports a mockable `DnsTxtLookup` interface so implementations can be tested without real DNS infrastructure. In production, the system DNS resolver is used by default.

---

## Frame Types

### AnnounceFrame (0x30)

A Node or Agent broadcasts its presence and capabilities. Receivers cache the announcement for `ttl` seconds. A `ttl` of `0` is a shutdown notification — receivers MUST remove the entry from their cache.

**Key fields:**

| Field | Type | Description |
|-------|------|-------------|
| `nid` | string | Publisher NID |
| `addresses` | array | Physical address list; each entry: `{host, port, protocol}` |
| `capabilities` | array | Capability list (reuses NIP capability vocabulary) |
| `ttl` | uint32 | Cache validity in seconds; `0` = offline/shutdown |
| `timestamp` | string | Broadcast time (ISO 8601 UTC) |
| `node_roles` | array of strings | All roles this publisher carries (see below) |
| `activation_mode` | string | `ephemeral` / `resident` / `hybrid` (see Activation Semantics below) |
| `activation_endpoint` | object | Push target for `resident` / `hybrid` publishers; same shape as `addresses[]` entry. REQUIRED when `activation_mode` is `resident` or `hybrid` |
| `cluster_anchor` | string (NID) | For non-Anchor nodes joining a cluster: identifies the Anchor Node they register with. Absent for standalone nodes and Anchor Nodes themselves. (NPS-CR-0001) |
| `bridge_protocols` | array of strings | For Bridge Nodes: supported external protocols (see Bridge Node section). MUST be absent for non-Bridge nodes. (NPS-CR-0001) |
| `spawn_spec_ref` | string | Opaque reference for constructing an Agent process on demand (ephemeral/hybrid cold start; standardized at NPS-Node Profile L3) |
| `signature` | string | Ed25519 signature with the publisher's IdentFrame private key — prevents announcement forgery |

### node_roles Field (renamed from node_kind in NDP v0.6)

`node_roles` is an array of strings carrying all roles this publisher supports. Valid values: `"memory"`, `"action"`, `"complex"`, `"anchor"`, `"bridge"`. Single-role nodes send a one-element array.

The legacy value `"gateway"` was removed in v1.0-alpha.3 (NPS-CR-0001). Parsers MUST reject it with `NDP-ANNOUNCE-ROLE-REMOVED`. Any other unrecognized value MUST be rejected with `NDP-ANNOUNCE-ROLE-UNKNOWN`.

**Backward compatibility:** NDP v0.6 renamed `node_kind` to `node_roles`. Parsers MUST accept the old field name `node_kind` as a parse-time alias through the alpha transition window. Pre-alpha.3 publishers that omit the field entirely are treated as declaring a single role matching `node_type`.

**Cross-protocol constraint:** the NWM `node_type` (single string, NWP service layer) MUST be one of the values declared in `node_roles`. Validators SHOULD verify this against cached NDP data.

### Activation Semantics

`activation_mode` tells receivers how to deliver frames to this publisher:

| Mode | Sender behavior | Receiver expectation |
|------|-----------------|----------------------|
| `ephemeral` | Deliver via inbox + pull; publisher may not be running when frame arrives | Frame is queued in the publisher's per-NID inbox |
| `resident` | Push over long-lived connection to `activation_endpoint` | Publisher accepts pushed frames on the declared endpoint at all times |
| `hybrid` | Attempt push to `activation_endpoint` first; fall back to inbox if unreachable within wake budget | Publisher wakes from hibernation on first frame; push target resumes once awake |

Backward compatibility: NPS v1.0-alpha.2 publishers did not emit `activation_mode`. Receivers MUST treat an absent field as `ephemeral`. A receiver MUST NOT reject an `AnnounceFrame` solely for lacking this field.

### Bridge Node: bridge_protocols (NPS-CR-0001)

Nodes that declare `"bridge"` in `node_roles` MUST also declare `bridge_protocols` — the list of external protocols this Bridge Node can translate to. Standard values:

| Value | External protocol |
|-------|------------------|
| `"http"` | HTTP / HTTPS (REST and streaming) |
| `"grpc"` | gRPC (unary and streaming) |
| `"mcp"` | Model Context Protocol |
| `"a2a"` | Agent-to-Agent protocol |

Third-party adapters MAY register additional values via future CRs. The `bridge_protocols` field MUST be absent for nodes that do not declare `"bridge"` in `node_roles`.

Bridge Nodes are stateless per request — they translate inbound NPS frames into the target protocol format and translate the response back into NPS frames (`CapsFrame`). They do not participate in cluster topology.

Direction note: Bridge Nodes translate NPS frames *outbound* to external protocols. The `compat/*-ingress` adapters (`mcp-ingress`, `a2a-ingress`, `grpc-ingress`) go in the opposite direction — they translate external protocol traffic *inbound* into NPS.

### Signed Announces: NdpAnnounceValidator

Every `AnnounceFrame` MUST carry a `signature` field — an Ed25519 signature by the publisher's NIP private key over the unsigned frame body (with `signature` field excluded), canonicalized per RFC 8785 JCS. This prevents announcement forgery: an attacker cannot claim to be a NID they do not control.

Receivers MUST verify the signature before caching or acting on an `AnnounceFrame`. Verification failure returns `NDP-ANNOUNCE-SIGNATURE-INVALID`. The `NdpAnnounceValidator` pattern (implemented in the .NET SDK as `NPS.NDP.Validation.NdpAnnounceValidator`) encapsulates signature verification plus NID-key binding check.

---

### ResolveFrame (0x31)

Resolves an `nwp://` URL to a physical endpoint. The request carries the `target` URL and an optional `requester_nid` for authorization checks. The response carries the resolved `{host, port, cert_fingerprint, ttl}`.

If the target cannot be resolved across all available modes, the resolver returns `NDP-RESOLVE-NOT-FOUND`. If multiple registries return conflicting results, `NDP-RESOLVE-AMBIGUOUS` is returned.

### GraphFrame (0x32)

Topology synchronization between NDP registries. Used for registry-to-registry gossip and change subscriptions. Carries a `seq` (monotonically increasing graph version), and either a full `nodes` array (`initial_sync: true`) or a JSON Patch (`initial_sync: false`) for incremental updates. The `seq` must be contiguous; gaps return `NDP-GRAPH-SEQ-GAP`.

---

## Agent Initialization Flow

```
Agent starts
  |
  +-- 1. NIP: load IdentFrame (from file or fetch from CA)
  |
  +-- 2. NDP: send AnnounceFrame (broadcast agent online)
  |
  +-- 3. NDP: subscribe to GraphFrame (receive node topology)
  |         <- GraphFrame(initial_sync=true)   [full graph]
  |         <- GraphFrame(initial_sync=false)  [incremental updates, ongoing]
  |
  +-- Ready: begin NWP requests
```

---

## Relationship to NWP Topology Queries (NPS-CR-0002)

NDP is the data store and delivery mechanism for cluster membership. NWP §12 is the *query surface* over that data.

When a node joins a cluster, it sends an `AnnounceFrame` with `cluster_anchor` naming its Anchor Node's NID. The Anchor Node receives this announcement and adds the member to its internal topology registry. That registry is then exposed via the reserved NWP query types `topology.snapshot` and `topology.stream` (see [Protocol NWP](Protocol-NWP)).

In other words: NDP feeds topology data into the Anchor Node; NWP §12 is how clients read it back out.

---

## Error Codes

| Error Code | NPS Status | Description |
|------------|------------|-------------|
| `NDP-RESOLVE-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | `nwp://` address could not be resolved across any available mode |
| `NDP-RESOLVE-AMBIGUOUS` | `NPS-CLIENT-CONFLICT` | Conflicting resolution results from multiple registries |
| `NDP-RESOLVE-TIMEOUT` | `NPS-SERVER-TIMEOUT` | Resolution request timed out |
| `NDP-ANNOUNCE-SIGNATURE-INVALID` | `NPS-AUTH-UNAUTHENTICATED` | AnnounceFrame signature verification failed |
| `NDP-ANNOUNCE-NID-MISMATCH` | `NPS-CLIENT-BAD-FRAME` | NID in AnnounceFrame does not match the signing certificate |
| `NDP-ANNOUNCE-ROLE-REMOVED` | `NPS-CLIENT-BAD-FRAME` | `node_roles` contains the retired `"gateway"` value (NPS-CR-0001); response SHOULD include a `hint` pointing to NPS-CR-0001 |
| `NDP-ANNOUNCE-ROLE-UNKNOWN` | `NPS-CLIENT-BAD-FRAME` | `node_roles` contains an unrecognized value |
| `NDP-GRAPH-SEQ-GAP` | `NPS-STREAM-SEQ-GAP` | GraphFrame sequence numbers are not contiguous |
| `NDP-REGISTRY-UNAVAILABLE` | `NPS-SERVER-UNAVAILABLE` | NDP Registry temporarily unavailable |

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
