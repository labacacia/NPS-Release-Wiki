# SDK Quickstart

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

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
| .NET / C# | `dotnet add package LabAcacia.NPS.Core --version 1.0.0-alpha.5.2` | `1.0.0-alpha.5.2` |
| Python | `pip install nps-lib==1.0.0a5.2` | `1.0.0a5.2` |
| TypeScript / Node | `npm install @labacacia/nps-sdk@1.0.0-alpha.5.2` | `1.0.0-alpha.5.2` |
| Java | `implementation("com.labacacia.nps:nps-java:1.0.0-alpha.5.2")` | `1.0.0-alpha.5.2` |
| Rust | `nps-sdk = "=1.0.0-alpha.5.2"` | `=1.0.0-alpha.5.2` (exact pin) |
| Go | `go get github.com/labacacia/NPS-sdk-go@v1.0.0-alpha.5.2` | `v1.0.0-alpha.5.2` |

> **Python package name:** The PyPI distribution name is `nps-lib` (not `nps-sdk` — that name is taken by an unrelated package). The Python import namespace is `nps_sdk`.

> **Rust pinning:** Use the `=` prefix for alpha releases to prevent Cargo from silently upgrading to a later alpha.

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

`AssuranceLevel.from_wire("")` (Python), `AssuranceLevel.fromWire("")` (TypeScript, Java), and equivalent calls in other SDKs must return `ANONYMOUS` — not raise an exception. This was a bug fixed in alpha.5. If you are on an older pin and see `ValueError` or `Unknown` for empty assurance levels, upgrade to `1.0.0-alpha.5.2`.

### Mixing suite versions

All NuGet/PyPI/npm/Maven/crates.io packages within the same language SDK are versioned together. Using `LabAcacia.NPS.Core 1.0.0-alpha.5` alongside `LabAcacia.NPS.NWP 1.0.0-alpha.5.2` is unsupported. Pin the whole suite to one version tag.

---

## What to read next

- Per-language full reference: [SDK Python](SDK-Python) · [SDK TypeScript](SDK-TypeScript) · [SDK Java](SDK-Java) · [SDK Rust](SDK-Rust) · [SDK Go](SDK-Go) · [SDK DotNet](SDK-DotNet)
- Setting up agent identities (Ed25519 NID, IdentFrame, trust chain): [SDK Identity and Authentication](SDK-Identity-and-Authentication)
- Building a production Anchor Node with middleware: [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)
- Common patterns (streaming, retries, anchor cache, DNS TXT fallback): [SDK Common Patterns](SDK-Common-Patterns)

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
