# SDK — .NET / C#

**Status:** ✅ Latest published packages — v1.0.0-alpha.14

C# / .NET 10 reference implementation for the Neural Protocol Suite. The .NET SDK is the canonical reference implementation for the suite — all spec changes are validated here first.

---

## NuGet packages

| Package | Version | Description |
|---------|---------|-------------|
| `LabAcacia.NPS.Core` | 1.0.0-alpha.14 | Shared frame types (`AnchorFrame`, `DiffFrame`, `StreamFrame`, `CapsFrame`, `HelloFrame`, `ErrorFrame`, `NopFrame` keepalive/heartbeat), JSON/MsgPack codecs, `AnchorFrameCache`, `FrameRegistry`; NCP native-mode transport (`NcpNativeClient`/`NcpServer`/`NcpSession`, added in alpha.11) |
| `LabAcacia.NPS.NWP` | 1.0.0-alpha.14 | Neural Web Protocol — NWM manifest, `QueryFrame`/`ActionFrame`/`SubscribeFrame`/`DiffFrame`, Memory/Action/Complex node middleware |
| `LabAcacia.NPS.NWP.Anchor` | 1.0.0-alpha.14 | NWP Anchor Node: stateless AaaS entry point translating `ActionFrame`s to NOP `TaskFrame`s; `AnchorNodeMiddleware`, `AnchorActionSpec`, `AnchorNodeClient` for `topology.snapshot` / `topology.stream` |
| `LabAcacia.NPS.NWP.Bridge` | 1.0.0-alpha.14 | NWP Bridge Node: stateless translator from NPS frames to non-NPS protocols (HTTP / gRPC / MCP / A2A target adapters) |
| `LabAcacia.NPS.NIP` | 1.0.0-alpha.14 | Neural Identity Protocol — CA, Ed25519 key generation, `IdentFrame` issuance/revocation, OCSP, CRL; X.509 + ACME `agent-01` challenge (RFC-0002) |
| `LabAcacia.NPS.NDP` | 1.0.0-alpha.14 | Neural Discovery Protocol — announce/resolve frames (`AnnounceFrame.spawn_spec_ref` structured schema object, `heartbeat_interval_ms`), in-memory registry, Ed25519 validation; DNS TXT fallback (`ResolveViaDns`, `IDnsTxtLookup`, `SystemDnsTxtLookup`) |
| `LabAcacia.NPS.NOP` | 1.0.0-alpha.14 | Neural Orchestration Protocol — `TaskFrame` (incl. `result_ttl_seconds`)/`DelegateFrame`/`SyncFrame`/`AlignStreamFrame`, DAG validator, orchestration engine |

**Requirements:** .NET 10 (LTS). All packages enable `<Nullable>enable</Nullable>`. MsgPack serialization uses `MessagePack 3.1.7` (alpha.13; the alpha.12 release was withdrawn for shipping the vulnerable `MessagePack 3.0.300` / NU1903).

**Tests:** 696 passing (.NET reference SDK, full protocol coverage).

> **Native-mode transport (RFC-0006), since alpha.11:** `NcpNativeClient` / `NcpServer` / `NcpSession` provide TCP length-prefix framing for NCP channels (`HelloFrame` on stream 0). This is the .NET reference for the native transport.

> **alpha.14 release delta:** The source tree now documents typed remote NIP CA clients, native-mode NWP serving helpers, TC-N1/TC-N2 conformance helpers, live revocation hooks, and native NCP TLS/mTLS hardening. NuGet install examples are pinned to alpha.14.

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

## See also

- [SDK Quickstart](SDK-Quickstart) — language-agnostic first steps and install table
- [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node) — full Anchor Node walkthrough using the .NET middleware
- [SDK Identity and Authentication](SDK-Identity-and-Authentication) — NipIdentity, IdentFrame, trust chain, NIP CA setup

---

*Last reviewed for published packages: v1.0.0-alpha.14*
