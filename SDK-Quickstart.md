# SDK Quickstart

**Status:** ✅ Latest published packages — v1.0.0-alpha.16

> **Audience:** Developers building Agents or Nodes against NPS for the first time.
> **Time to first frame:** 10–15 minutes.
> **Source-of-truth precedence:** `spec/` documents in the NPS-Release repository win over this page if they disagree.

NPS (Neural Protocol Suite) is a five-protocol stack designed for AI agent communication. Before writing any code, it helps to understand why the first exercise below is what it is.

---

## Why AnchorFrame is the right first exercise

An **AnchorFrame** is a content-addressed schema descriptor. Instead of embedding field names and types in every response, a node publishes one AnchorFrame (identified by `sha256:<hash>`); subsequent data frames carry only the anchor reference. This is the mechanism that cuts token consumption by 30–60% compared to HTTP/REST. Every other frame type in NPS either references an anchor or builds on the same codec infrastructure. Learning to encode and decode an AnchorFrame means you have learned the codec, the registry, and the encoding tiers — which are the same primitives used for QueryFrame, TaskFrame, and everything else.

---

## Installation

Pin the entire suite to a single version. Mixing patch versions within the same suite version is not supported.

| Language | Install command | Current pin |
|----------|-----------------|-------------|
| .NET / C# | `dotnet add package LabAcacia.NPS.Core --version 1.0.0-alpha.16` | `1.0.0-alpha.16` |
| Python | `pip install nps-lib==1.0.0a16` | `1.0.0a16` |
| TypeScript / Node | `npm install @labacacia/nps-sdk@1.0.0-alpha.16` | `1.0.0-alpha.16` |
| Java | `implementation("com.labacacia.nps:nps-java:1.0.0-alpha.16")` | `1.0.0-alpha.16` |
| Rust | `nps-sdk = "=1.0.0-alpha.16"` | `=1.0.0-alpha.16` (exact pin) |
| Go | `go get github.com/labacacia/NPS-sdk-go@v1.0.0-alpha.16` | `v1.0.0-alpha.16` |

> **Python package name:** The PyPI distribution name is `nps-lib` (not `nps-sdk` — that name is taken by an unrelated package). The Python import namespace is `nps_sdk`.

> **Rust pinning:** Use the `=` prefix for alpha releases to prevent Cargo from silently upgrading to a later alpha.

> **npm `alpha` dist-tag:** `@labacacia/nps-sdk@alpha` currently resolves to `1.0.0-alpha.16`. Pin the explicit version above for reproducible builds.

> **Release note:** alpha.12 was withdrawn (vulnerable `MessagePack 3.0.300` / NU1903 plus a native-mode handshake bug). alpha.13 superseded it with `MessagePack 3.1.7`; alpha.15 is the current pin.

> **alpha.15 release docs:** Source and specs now cover NCP Tier-3 BinaryVector (`binary_vector.v1`) compact float-vector encoding, inbound NWP Bridge server adapters (external MCP / A2A clients calling local NPS actions), native-mode NWP serving, and a typed remote NIP CA client — plus the earlier alpha.14 additions (typed CA clients, native-mode serving helpers, TC-N1/TC-N2 conformance helpers, live revocation hooks, .NET native NCP TLS/mTLS hardening). alpha.15 also realigns the NIP TrustFrame/RevokeFrame signed payload (**breaking:** old signed frames no longer verify).

---

## The 3-step pattern

Every SDK follows the same pattern regardless of language:

```
# Step 1 — create a codec and frame registry
registry = create_full_registry()
codec    = NpsFrameCodec(registry)

# Step 2 — build a frame
frame = AnchorFrame(
    anchor_id = "sha256:abc123",
    schema    = { "fields": [{ "name": "price", "type": "decimal" }] },
    ttl       = 3600
)

# Step 3 — encode to wire bytes, then decode back
wire   = codec.encode(frame)          # Tier-2 MsgPack by default
result = codec.decode(wire)           # returns AnchorFrame
```

This pseudo-code maps directly to real method names in every SDK. See the per-language pages for exact import paths and syntax.

### Encoding tiers

| Tier | Wire value | Use case |
|------|------------|----------|
| Tier-1 JSON | `0x00` | Development, debugging, HTTP interop |
| Tier-2 MsgPack | `0x01` | **Production default** — ~60% smaller than JSON |

Always use MsgPack (Tier-2) in production. The codec defaults to MsgPack unless you explicitly override the tier.

---

## First exercise: encode an AnchorFrame and decode it back

The goal is to verify your install and confirm the round-trip is lossless.

### Python

```python
from nps_sdk.core.codec import NpsFrameCodec
from nps_sdk.core.registry import FrameRegistry
from nps_sdk.ncp.frames import AnchorFrame, FrameSchema, SchemaField

registry = FrameRegistry.create_default()
codec    = NpsFrameCodec(registry)

schema = FrameSchema(fields=(
    SchemaField(name="id",    type="uint64"),
    SchemaField(name="price", type="decimal", semantic="commerce.price.usd"),
))
frame  = AnchorFrame(anchor_id="sha256:abc123", schema=schema)

wire   = codec.encode(frame)    # bytes — MsgPack
result = codec.decode(wire)     # AnchorFrame
assert result.anchor_id == frame.anchor_id
```

### TypeScript

```typescript
import { NpsFrameCodec, createDefaultRegistry } from "@labacacia/nps-sdk/core";
import { AnchorFrame } from "@labacacia/nps-sdk/ncp";

const codec = new NpsFrameCodec(createDefaultRegistry());
const frame = new AnchorFrame("sha256:abc123", {
  fields: [{ name: "price", type: "decimal" }]
});

const wire   = codec.encode(frame);
const result = codec.decode(wire);
```

### .NET / C#

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

byte[] wire   = codec.Encode(frame);
var    result = (AnchorFrame)codec.Decode(wire);
```

See per-language pages for Java, Rust, and Go equivalents.

---

## Common first mistakes

### Using `estimated_npt` instead of `cgn_est`

As of v1.0.0-alpha.5.1 / alpha.5.2, the wire field for token budget estimates was renamed from `estimated_npt` to `cgn_est` (Cognon, not Neural Processing Token). The C# property was also renamed: `EstimatedNpt` → `EstimatedCgn`. If you see null values in topology events or action specs, check that you are reading `cgn_est` and not the old key.

### Using JSON tier in production

Tier-1 JSON is convenient for debugging but produces roughly 2.5× more bytes than Tier-2 MsgPack. The codec defaults to MsgPack. If you explicitly pass `EncodingTier.JSON` (or equivalent), switch it back before load testing.

### Ignoring the `AssuranceLevel` empty-string case

`AssuranceLevel.from_wire("")` (Python), `AssuranceLevel.fromWire("")` (TypeScript, Java), and equivalent calls in other SDKs must return `ANONYMOUS` — not raise an exception. This was a bug fixed in alpha.5. If you are on an older pin and see `ValueError` or `Unknown` for empty assurance levels, upgrade to `1.0.0-alpha.16`.

### Mixing suite versions

All NuGet/PyPI/npm/Maven/crates.io packages within the same language SDK are versioned together. Using `LabAcacia.NPS.Core 1.0.0-alpha.5` alongside `LabAcacia.NPS.NWP 1.0.0-alpha.5.2` is unsupported. Pin the whole suite to one version tag.

---

## Published alpha.15 feature set

All six SDKs (Python, TypeScript, Go, Java, Rust, .NET) ship the alpha.13 parity surface plus the alpha.14 and alpha.15 release additions:

- **NCP** — `NopFrame` (0x07) zero-payload keepalive/heartbeat; `HelloFrame.ping_interval_ms` (uint32, 0 = disabled; dead-peer threshold = 3 × interval).
- **NIP** — `IdentFrame.node_roles` (self-declared node-role tags, excluded from the Ed25519-signed payload).
- **NDP** — `AnnounceFrame.spawn_spec_ref` as a structured schema object (was a URI string); `AnnounceFrame.heartbeat_interval_ms` (uint32, default 60000 ms, 0 = disabled).
- **NOP** — `TaskFrame.result_ttl_seconds` (uint32, default 3600 s, omitted from wire at default).
- **NWP** — `X-NWM-Version` HTTP response header; NWM `manifest_version` (uint32 monotonic counter) and `manifest_updated_at` (ISO 8601).
- `ReputationLogClient` (CT-style reputation log, dual Ed25519 signatures) has been available across all six SDKs since **alpha.7**.
- The .NET SDK additionally ships NCP **native-mode transport** (`NcpNativeClient` / `NcpServer` / `NcpSession`) since **alpha.11**.

The alpha.14 release adds:

- **Remote NIP CA clients** — typed client surfaces for issue, renew, revoke, CRL, and OCSP flows.
- **Native NWP serving helpers** — SDK helpers for serving NWP nodes over native NCP sessions, rather than only HTTP middleware.
- **Conformance helpers** — TC-N1/TC-N2 manifests and harness entry points for repeatable SDK/spec checks.
- **.NET hardening** — live revocation hooks plus native NCP TLS/mTLS hooks and handshake bounds.

The alpha.15 release adds (capability described here; the exact API names are .NET — see [SDK DotNet](SDK-DotNet); other SDKs expose the same capability under their own naming):

- **NCP Tier-3 BinaryVector** (`binary_vector.v1`) — a third encoding tier for compact float-vector (embedding) payloads on `QueryFrame.vector_search.vector`. Negotiated via caps and used only when both peers advertise `binary_vector.v1`; `NPBV`-prefixed payload with MessagePack metadata markers and little-endian `float32` vector segments. Malformed payloads return documented client errors (`NCP-BINARY-VECTOR-*` → `NPS-CLIENT-BAD-FRAME`); the reserved tier `0b11` returns `NCP-FRAME-FLAGS-INVALID`.
- **Inbound NWP Bridge server adapters** — let external MCP / A2A clients call local NPS actions (the inverse of the outbound Bridge Node). Secure-by-default: a valid `X-NWP-Agent` NID plus a verifier hook, an action allowlist, bounded request bodies (default 1 MB → 413), a dispatch timeout (default 30 s → 504), and sanitized client errors.
- **Native-mode NWP serving** — Memory / Action Nodes serve `QueryFrame` / `ActionFrame` directly over a native NCP session.
- **Typed remote NIP CA client** — CA discovery, CRL retrieval, and Ed25519 register / renew / revoke / verify, including RFC-0002 X.509 registration.
- **NIP TrustFrame/RevokeFrame signed-payload realignment** — the Ed25519-signed payload now includes the current NPS-3 fields (`issued_at`, `serial`, `signer_nid`, `target_nid`) and uses current revocation naming (`NIP-CERT-REVOKED`). **Breaking:** signed frames produced by the old alpha.14-era SDK shape no longer verify after upgrading.

---

## What to read next

- Per-language full reference: [SDK Python](SDK-Python) · [SDK TypeScript](SDK-TypeScript) · [SDK Java](SDK-Java) · [SDK Rust](SDK-Rust) · [SDK Go](SDK-Go) · [SDK DotNet](SDK-DotNet)
- Setting up agent identities (Ed25519 NID, IdentFrame, trust chain): [SDK Identity and Authentication](SDK-Identity-and-Authentication)
- Building a production Anchor Node with middleware: [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)
- Common patterns (streaming, retries, anchor cache, DNS TXT fallback): [SDK Common Patterns](SDK-Common-Patterns)

---

*Last reviewed for published packages: v1.0.0-alpha.16*

---

*Last reviewed at suite version: v1.0.0-alpha.16*
