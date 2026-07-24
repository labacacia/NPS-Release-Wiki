# SDK — .NET / C#

**Status:** ✅ Latest published packages — v1.0.0-alpha.16

C# / .NET 10 reference implementation for the Neural Protocol Suite. The .NET SDK is the canonical reference implementation for the suite — all spec changes are validated here first.

---

## NuGet packages

| Package | Version | Description |
|---------|---------|-------------|
| `LabAcacia.NPS.Core` | 1.0.0-alpha.16 | Shared frame types (`AnchorFrame`, `DiffFrame`, `StreamFrame`, `CapsFrame`, `HelloFrame`, `ErrorFrame`, `NopFrame` keepalive/heartbeat), JSON/MsgPack/BinaryVector codecs, `AnchorFrameCache`, `FrameRegistry`; NCP native-mode transport (`NcpNativeClient`/`NcpServer`/`NcpSession`, added in alpha.11); NCP Tier-3 BinaryVector codec (`Tier3BinaryVectorCodec`, alpha.14) |
| `LabAcacia.NPS.NWP` | 1.0.0-alpha.16 | Neural Web Protocol — NWM manifest, `QueryFrame`/`ActionFrame`/`SubscribeFrame`/`DiffFrame`, Memory/Action/Complex node middleware; native-mode serving (`NwpNativeNodeServer`); inbound Bridge server adapters (`AddBridgeServer`/`UseBridgeServer`, `McpServerBridge`/`A2aServerBridge`) |
| `LabAcacia.NPS.NWP.Anchor` | 1.0.0-alpha.16 | NWP Anchor Node: stateless AaaS entry point translating `ActionFrame`s to NOP `TaskFrame`s; `AnchorNodeMiddleware`, `AnchorActionSpec`, `AnchorNodeClient` for `topology.snapshot` / `topology.stream` |
| `LabAcacia.NPS.NWP.Bridge` | 1.0.0-alpha.16 | NWP Bridge Node: stateless translator from NPS frames to non-NPS protocols (HTTP / gRPC / MCP / A2A target adapters) |
| `LabAcacia.NPS.NIP` | 1.0.0-alpha.16 | Neural Identity Protocol — CA, Ed25519 key generation, `IdentFrame` issuance/revocation, OCSP, CRL; X.509 + ACME `agent-01` challenge (RFC-0002); typed remote CA client (`NipCaClient`) |
| `LabAcacia.NPS.NDP` | 1.0.0-alpha.16 | Neural Discovery Protocol — announce/resolve frames (`AnnounceFrame.spawn_spec_ref` structured schema object, `heartbeat_interval_ms`), in-memory registry, Ed25519 validation; DNS TXT fallback (`ResolveViaDns`, `IDnsTxtLookup`, `SystemDnsTxtLookup`) |
| `LabAcacia.NPS.NOP` | 1.0.0-alpha.16 | Neural Orchestration Protocol — `TaskFrame` (incl. `result_ttl_seconds`)/`DelegateFrame`/`SyncFrame`/`AlignStreamFrame`, DAG validator, orchestration engine |

**Requirements:** .NET 10 (LTS). All packages enable `<Nullable>enable</Nullable>`. MsgPack serialization uses `MessagePack 3.1.7` (alpha.13; the alpha.12 release was withdrawn for shipping the vulnerable `MessagePack 3.0.300` / NU1903).

**Tests:** 696 passing (.NET reference SDK, full protocol coverage).

> **Native-mode transport (RFC-0006), since alpha.11:** `NcpNativeClient` / `NcpServer` / `NcpSession` provide TCP length-prefix framing for NCP channels (`HelloFrame` on stream 0). This is the .NET reference for the native transport.

> **alpha.16 release delta:** NWP LLM/Thinking Profile support — official DTOs/helpers for `profiles.llm` (model descriptors, streaming/tool support, privacy hints, reasoning-disclosure policy) and the typed `llm.complete` Action/Caps/Stream contracts; canonical NWP HTTP-binding rejection error codes; NIP RA stores persisted in the CA storage backends. Carries the full alpha.15 delta (NCP Tier-3 BinaryVector, inbound NWP Bridge server adapters, `NwpNativeNodeServer`, `NipCaClient`; **breaking:** NIP TrustFrame/RevokeFrame signed-payload realignment — alpha.14-era signed frames no longer verify). NuGet install examples are pinned to alpha.16. See the [alpha.15 feature set](#alpha15-feature-set) below for the accumulated surface.

---

## DI registration

The SDK is designed for ASP.NET Core. Each layer has a `services.Add*()` extension and an `app.Use*()` pipeline hook.

### Minimal stack (Core only)

```csharp
// Program.cs
builder.Services.AddNpsCore(opts =>
{
    opts.DefaultTier        = EncodingTier.MsgPack; // production default
    opts.AnchorTtlSeconds   = 3600;
    opts.AllowPlaintext     = false;                // set true only in dev
});
```

`AddNpsCore` registers: `FrameRegistry` (singleton), `Tier1JsonCodec` (singleton), `Tier2MsgPackCodec` (singleton), `NpsFrameCodec` (singleton), `AnchorFrameCache` (scoped — per connection).

### NWP layer

```csharp
builder.Services.AddNpsCore();
builder.Services.AddNwp(opts =>
{
    opts.Port              = 17433;
    opts.DefaultTokenBudget = 0;   // 0 = no limit
    opts.MaxDepth          = 5;
    opts.DefaultLimit      = 20;
});
```

### Memory Node

```csharp
builder.Services.AddNpsCore();
builder.Services.AddNwp();
builder.Services.AddMemoryNode<MyProvider>(opts =>
{
    opts.NodeId     = "urn:nps:node:example.com:data";
    opts.Schema     = myAnchorFrame;
    opts.PathPrefix = "/data";
});

// In the pipeline:
app.UseMemoryNode<MyProvider>(opts => { ... });
```

### Action Node

```csharp
builder.Services.AddActionNode<MyActionProvider>(opts =>
{
    opts.NodeId  = "urn:nps:node:example.com:actions";
    opts.Actions = new Dictionary<string, ActionSpec>
    {
        ["orders.create"] = new ActionSpec { ... }
    };
});

app.UseActionNode<MyActionProvider>(opts => { ... });
```

### Anchor Node

```csharp
// Requires a previously registered INopOrchestrator
builder.Services.AddNopOrchestrator(...);
builder.Services.AddAnchorNode<MyAnchorRouter>(opts =>
{
    opts.NodeId                   = "urn:nps:node:example.com:anchor";
    opts.PathPrefix               = "/anchor";
    opts.RequireTopologyCapability = false;   // set true to gate topology queries
    opts.Actions = new Dictionary<string, AnchorActionSpec>
    {
        ["summarise"] = new AnchorActionSpec
        {
            Async       = true,
            EstimatedCgn = 200,   // hint advertised in NWM
        }
    };
});

// Optional: in-memory topology service (L2 conformance)
builder.Services.AddInMemoryAnchorTopology("urn:nps:node:example.com:anchor");

app.UseAnchorNode(opts => { ... });
```

### NIP CA

```csharp
builder.Services.AddNipCa(opts =>
{
    opts.CaNid         = "urn:nps:ca:example.com";
    opts.KeyFilePath   = "/run/secrets/ca.key";
    opts.KeyPassphrase = Environment.GetEnvironmentVariable("NIP_CA_PASSPHRASE")!;
    opts.BaseUrl       = "https://ca.example.com";
    opts.AcmeEnabled   = true;
}, generateKeyIfMissing: false);

app.MapNipCa();
app.UseNipAcme();
```

> **`SqliteNipCaStore`** — new in v1.0.0-alpha.5.2: an embedded CA store backed by SQLite, suitable for single-node deployments that do not need PostgreSQL. Use it by registering `INipCaStore` as `SqliteNipCaStore` before calling `AddNipCa`.

### NIP Verifier (node-side)

```csharp
builder.Services.AddNipVerifier(opts =>
{
    opts.TrustedIssuers = new Dictionary<string, string>
    {
        ["urn:nps:ca:example.com"] = "ed25519:<hex-public-key>"
    };
});
```

---

## Key classes

### `NpsFrameCodec`

```csharp
using NPS.Core.Codecs;
using NPS.Core.Registry;
using NPS.Core.Frames.Ncp;

var registry = FrameRegistry.CreateDefault();
var codec    = new NpsFrameCodec(registry);

var frame = new AnchorFrame
{
    AnchorId = "sha256:abc123",
    Schema   = new { fields = new[] { new { name = "price", type = "decimal" } } },
    Ttl      = 3600
};

byte[] wire   = codec.Encode(frame);              // Tier-2 MsgPack by default
var    result = (AnchorFrame)codec.Decode(wire);
```

### `AnchorActionSpec`

Describes one action exposed by an Anchor Node in its NWM (Neural Web Manifest):

```csharp
var spec = new AnchorActionSpec
{
    Description  = "Summarise the input document",
    ParamsAnchor = "sha256:params-schema-id",
    ResultAnchor = "sha256:result-schema-id",
    Async        = true,
    EstimatedCgn = 200,          // cgn_est wire key; post-alpha.5.1 rename from EstimatedNpt
    TimeoutMsDefault = 30_000,
    RequiredCapability = "nwp:invoke",
};
```

The wire key is `cgn_est` (renamed from `estimated_npt` in v1.0.0-alpha.5.2). Clients on an older pin will see null for this field.

### `AnchorNodeMiddleware`

Stateless ASP.NET Core middleware implementing an Anchor Node (NPS-AaaS §2). It:
- Validates auth and rate limits on `/invoke`
- Dispatches `ActionFrame` to the registered `IAnchorRouter`
- Fires the resulting `TaskFrame` to the local `INopOrchestrator`
- Returns synchronous results as `CapsFrame`; async returns HTTP 202 with `task_id`
- Gates `topology.snapshot` / `topology.stream` behind `topology:read` capability when `RequireTopologyCapability = true`
- Returns HTTP 501 / `NPS-SERVER-UNSUPPORTED` for unrecognised reserved frame types

### `NopFrame` — keepalive / heartbeat (alpha.13)

NCP frame type `0x07`, a zero-payload keepalive. After the handshake either peer MAY send `NopFrame`s. The cadence is advertised through `HelloFrame.ping_interval_ms` (uint32; `0` disables keepalive); the dead-peer threshold is `3 × ping_interval_ms`. A missed keepalive surfaces as `NCP-KEEPALIVE-TIMEOUT` (mapped to `NPS-SERVER-TIMEOUT`).

```csharp
var hello = new HelloFrame
{
    PingIntervalMs = 15_000,   // send/expect a NopFrame every 15 s; dead-peer at 45 s
};
```

### `AssuranceLevel` — empty-string case

The `AssuranceLevel` enum parses the NIP RFC-0003 wire string. An empty string returns `Anonymous`:

```csharp
using NPS.NIP;

var level = AssuranceLevel.Parse("");    // → AssuranceLevel.Anonymous
var level2 = AssuranceLevel.Parse("L1"); // → AssuranceLevel.L1
```

The implementation uses `string.IsNullOrEmpty(wire)` to guard the empty case.

---

## `cgn_est` rename (breaking change in alpha.5.2)

The wire field on `AnchorActionSpec` changed from `estimated_npt` to `cgn_est`. The C# property was renamed in alpha.5.1 (`EstimatedNpt` → `EstimatedCgn`); the wire key was renamed in alpha.5.2. There is no alias — old clients reading `estimated_npt` will see null after upgrade. This aligns with the CGN (Cognon) naming convention used throughout the suite.

Related renames in the same package:
- `NptMeter` → `CognMeter` (static utility class in `NPS.NWP`)
- `BudgetNpt`, `AvailableNpt` → `BudgetCgn`, `AvailableCgn` (in `AnchorNodeMiddleware` internals)

---

## DNS TXT fallback (NDP)

```csharp
using NPS.NDP;

var registry  = new InMemoryNdpRegistry();
var dnsLookup = new SystemDnsTxtLookup(); // uses DnsClient v1.8.0

// Falls back to _nps-node.<host> TXT lookup when no in-memory entry matches
var result = await registry.ResolveViaDns("nwp://example.com/data/items", dnsLookup);
```

Inject a custom `IDnsTxtLookup` implementation for testing.

---

## Building and testing

```bash
# Build all packages
dotnet build NPS.sln

# Run all tests (xunit.v3)
dotnet test
```

---

## alpha.15 feature set

The reference SDK ships the alpha.13 parity surface plus the alpha.14 and alpha.15 release additions. The API names below are the .NET reference surface.

### NCP Tier-3 BinaryVector (`binary_vector.v1`)

NCP **v0.9** activates the third encoding tier (`Flags.T1T0 = 0b10`), a compact AI-native encoding for vector-heavy frames. Metadata stays MessagePack; dense vector values are carried as raw little-endian `float32` segments. The standard binding is NWP `QueryFrame.vector_search.vector`.

- **Negotiated only.** Senders MUST NOT emit Tier-3 unless both peers advertised `binary_vector.v1` during the handshake. A receiver that did not negotiate Tier-3 rejects the frame with `NCP-ENCODING-UNSUPPORTED`. The reserved tier `Flags.T1T0 = 0b11` is rejected with `NCP-FRAME-FLAGS-INVALID`.
- **Payload layout.** A 16-byte prefix — `NPBV` magic (4 bytes), version `0x01`, flags byte (`0x00`), `vector_count` (uint16 BE), `metadata_len` (uint32 BE), 4 reserved zero bytes — followed by `metadata_len` bytes of MessagePack metadata (Tier-2 field names; vector fields replaced by a `{"$nps_binary_vector": <index>, "dtype": "float32", "dim": <n>}` marker), then per-vector segments of `dim` (uint32 BE) + `dim` little-endian `float32` values.
- **Malformed payloads return documented client errors**, never a server-internal: `NCP-BINARY-VECTOR-MALFORMED` (bad magic / structure), `-DIM-MISMATCH`, `-INDEX-INVALID`, `-DTYPE-UNSUPPORTED`, `-TRUNCATED` — all mapping to `NPS-CLIENT-BAD-FRAME`.

```csharp
// EncodingTier.BinaryVector maps to Flags.T1T0 = 0b10.
// Round-trips a QueryFrame.vector_search.vector through Tier3BinaryVectorCodec.
var wire = codec.Encode(queryFrame, EncodingTier.BinaryVector);
var back = (QueryFrame)codec.Decode(wire);
```

The native-mode handshake (`NcpNativeClient` / `NcpServer`) negotiates `binary_vector.v1` automatically when both peers advertise it.

### Inbound NWP Bridge server adapters

The inbound Bridge server lets **external** MCP / A2A clients call **local NPS actions** — the inverse of the outbound `BridgeNode` dispatchers (which translate NPS frames out to HTTP / gRPC / MCP / A2A targets and also landed in alpha.14). Adapters: `McpServerBridge` and `A2aServerBridge`, registered through the ASP.NET Core `AddBridgeServer` / `UseBridgeServer` extensions.

It is **secure-by-default** — every gate must pass before a request reaches a local action:

- requires a valid `X-NWP-Agent` NID plus a configured verifier hook;
- an explicit action **allowlist** (only listed actions are reachable);
- bounded request bodies (`MaxRequestBodyBytes`, default 1 MB; oversize → HTTP 413);
- a dispatch timeout (`DispatchTimeoutMs`, default 30 s; exceeded → HTTP 504);
- sanitized client error responses (internal detail is not leaked to the caller).

```csharp
builder.Services.AddBridgeServer(opts =>
{
    opts.MaxRequestBodyBytes = 1_048_576;   // 1 MB → 413 when exceeded
    opts.DispatchTimeoutMs   = 30_000;      // 30 s → 504 when exceeded
    opts.AllowedActions      = new[] { "orders.create", "summarise" };
    opts.AgentVerifier       = myXNwpAgentVerifier;   // validates the X-NWP-Agent NID
});

app.UseBridgeServer();   // mounts McpServerBridge / A2aServerBridge endpoints
```

### Native-mode NWP serving (`NwpNativeNodeServer`)

`NwpNativeNodeServer` lets Memory and Action Nodes serve `QueryFrame` / `ActionFrame` directly over an `NcpSession` / native NCP stream, instead of a hand-rolled frame loop or the HTTP middleware. This is the native-transport counterpart to `UseMemoryNode` / `UseActionNode`.

```csharp
// Serve a registered node provider directly over a native NCP session.
var server = new NwpNativeNodeServer(memoryNodeProvider, options);
await server.ServeAsync(ncpSession, cancellationToken);
```

### Typed remote NIP CA client (`NipCaClient`)

`NipCaClient` is a typed client for a remote NIP CA: CA discovery (`/.well-known/nps-ca`), CRL retrieval (`/v1/crl`, now including `issued_at` plus a detached CA signature), and Ed25519 register / renew / revoke / verify flows, including RFC-0002 X.509 registration. The CA store gains `INipCaStore.ListAsync()` with an `InMemoryNipCaStore` implementation; `/.well-known/nps-ca` no longer advertises an unmapped `/ocsp`.

```csharp
var ca = new NipCaClient("https://ca.example.com");
var discovery = await ca.DiscoverAsync();          // /.well-known/nps-ca
var issued    = await ca.RegisterAsync(identity);  // Ed25519 enrolment
var crl       = await ca.GetCrlAsync();            // includes issued_at + detached CA signature
```

### NIP TrustFrame / RevokeFrame signed-payload realignment (breaking)

The Ed25519-signed payload of `TrustFrame` and `RevokeFrame` now covers the current NPS-3 field set — including `issued_at`, `serial`, `signer_nid`, `target_nid`, where applicable — over the canonical JSON with the `signature` field removed (same canonicalisation rule as IdentFrame, §5.1/§5.2/§5.3), and uses the current revocation naming (`NIP-CERT-REVOKED`). **Breaking:** signed frames produced by the alpha.14-era SDK shape no longer verify after upgrading; re-issue trust and revocation frames with an alpha.15 signer.

### Daemon observability & conformance

- Transport-neutral `HealthProbeRenderer` for `/healthz` · `/readyz` probes.
- `LabAcacia.NPS.Conformance` package with the Node L1/L2 case catalogs (TC-N1/TC-N2 entry points).
- The NuGet family is 11 SDK packages plus 3 ingress packages (`McpIngress` / `A2aIngress` / `GrpcIngress`), all now published at alpha.15 (the ingress packages, deferred in alpha.13, are caught up).

---

## See also

- [SDK Quickstart](SDK-Quickstart) — language-agnostic first steps and install table
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node) — full Anchor Node walkthrough using the .NET middleware
- [SDK Identity and Authentication](SDK-Identity-and-Authentication) — NipIdentity, IdentFrame, trust chain, NIP CA setup

---

*Last reviewed for published packages: v1.0.0-alpha.16*

---

*Last reviewed at suite version: v1.0.0-alpha.16*
