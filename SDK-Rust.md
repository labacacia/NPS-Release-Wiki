# SDK — Rust

**Status:** ✅ Content complete — v1.0.0-alpha.14

Rust client library for the Neural Protocol Suite. Covers all five protocols: NCP, NWP, NIP, NDP, and NOP.

---

## Installation

Add to your `Cargo.toml`:

```toml
[dependencies]
nps-sdk = "=1.0.0-alpha.14"
tokio   = { version = "1", features = ["rt-multi-thread", "macros"] }
```

> **Pin with `=`:** For alpha releases, use the exact-version prefix (`=`) to prevent Cargo from silently upgrading to a later alpha. Semver pre-release rules do not guarantee backward compatibility across alphas.

**Requirements:** Rust stable (1.70+). No nightly features are required.

**Tests:** 119 passing across the workspace.

---

## Workspace crates

| Crate | Description |
|-------|-------------|
| `nps-core` | Frame header, `NpsFrameCodec` (Tier-1 JSON / Tier-2 MsgPack), `FrameRegistry`, anchor cache, `NpsError` |
| `nps-ncp` | NCP frames: `AnchorFrame`, `DiffFrame`, `StreamFrame`, `CapsFrame`, `HelloFrame` (`ping_interval_ms`), `NopFrame` (0x07 keepalive), `ErrorFrame` |
| `nps-nwp` | NWP frames: `QueryFrame`, `ActionFrame`, `AsyncActionResponse`; async `NwpClient` (reqwest; reads `X-NWM-Version` and `manifest_version`/`manifest_updated_at` for conditional `.nwm` re-fetch); `error_codes` module (30 constants) |
| `nps-nip` | NIP frames: `IdentFrame` (incl. `node_roles` self-declared role tags), `TrustFrame`, `RevokeFrame`; `NipIdentity` (Ed25519 key management); `ReputationLogClient` (RFC-0004 Phase 2, added in alpha.7); `nps_nip::x509` (anchored to IANA PEN 65715); `nps_nip::acme` |
| `nps-ndp` | NDP frames: `AnnounceFrame` (`spawn_spec_ref` structured schema object; `heartbeat_interval_ms`), `ResolveFrame`, `GraphFrame`; `InMemoryNdpRegistry`; `NdpAnnounceValidator`; `resolve_via_dns`, `DnsTxtLookup` trait, `parse_nps_txt_record` |
| `nps-nop` | NOP frames: `TaskFrame` (`result_ttl_seconds`), `DelegateFrame`, `SyncFrame`, `AlignStreamFrame`; `BackoffStrategy`; `NopClient` |
| `nps-sdk` | Re-export umbrella crate — all protocols under `nps_sdk::` namespace |

All crates are in the same Cargo workspace. You can depend on the umbrella `nps-sdk` crate or on individual crates if you only need specific protocols.

---

## Feature flags (`nps-sdk`)

| Feature | Default | Description |
|---------|---------|-------------|
| `nwp` | enabled | Include NWP frames and client |
| `nip` | enabled | Include NIP frames and identity |
| `ndp` | enabled | Include NDP frames, registry, validator |
| `nop` | enabled | Include NOP frames and client |

---

## Key structs and functions

### `NpsFrameCodec` — encode an AnchorFrame

```rust
use nps_core::codec::NpsFrameCodec;
use nps_core::frames::EncodingTier;
use nps_core::registry::FrameRegistry;
use nps_ncp::AnchorFrame;
use serde_json::json;

let codec = NpsFrameCodec::new(FrameRegistry::create_full());

let mut schema = serde_json::Map::new();
schema.insert("fields".into(), json!([{"name": "id", "type": "uint64"}]));

let frame = AnchorFrame {
    anchor_id: "sha256:abc123".into(),
    schema,
    namespace:   None,
    description: None,
    node_type:   None,
    ttl:         3600,
};

let wire = codec.encode(AnchorFrame::frame_type(), &frame.to_dict(), EncodingTier::MsgPack, true)?;
let (frame_type, dict) = codec.decode(&wire)?;
let back = AnchorFrame::from_dict(&dict)?;
```

### `NwpClient` (async, tokio)

```rust
use nps_nwp::{NwpClient, QueryFrame};

let client = NwpClient::new("http://node.example.com:17433");
let query  = QueryFrame::new("sha256:abc123");

// Query
let caps = client.query(&query).await?;

// Stream
let frames = client.stream(&query).await?;
for sf in &frames {
    println!("{:?}", sf.payload);
    if sf.is_last { break; }
}

// Action invoke
use nps_nwp::ActionFrame;
let af     = ActionFrame::new("summarise", serde_json::json!({"maxTokens": 500}));
let result = client.invoke(&af).await?;
```

The codec and the `AnchorFrame` encode/decode path are synchronous — no runtime required. Only the `NwpClient` and `NopClient` require tokio.

### `NipIdentity`

```rust
use nps_nip::identity::NipIdentity;
use std::path::Path;

// Generate keypair
let identity = NipIdentity::generate();
println!("{}", identity.pub_key_string()); // "ed25519:<hex>"

// Sign a payload
let mut payload = serde_json::Map::new();
payload.insert("nid".into(), serde_json::json!("urn:nps:node:example.com:data"));
let sig = identity.sign(&payload);        // "ed25519:<base64>"
let ok  = identity.verify(&payload, &sig); // true

// Persist and load (AES-256-GCM + PBKDF2)
identity.save(Path::new("my-node.key"), "my-passphrase")?;
let loaded = NipIdentity::load(Path::new("my-node.key"), "my-passphrase")?;
```

### DNS TXT fallback

```rust
use nps_ndp::resolve_via_dns;

let result = resolve_via_dns("nwp://example.com/data/items", None)?;
// result.host, result.port, result.protocol
```

The `DnsTxtLookup` trait allows injecting a custom resolver. Pass `None` to use the system DNS resolver.

---

## Error handling

All operations return `NpsResult<T>` which is `Result<T, NpsError>`. There are no panics in the hot path.

| Variant | When |
|---------|------|
| `NpsError::Frame(msg)` | Unknown frame type, invalid field |
| `NpsError::Codec(msg)` | Encode/decode failure, oversized payload |
| `NpsError::AnchorNotFound(id)` | `get_required()` for a missing/expired anchor |
| `NpsError::AnchorPoison(id)` | Attempt to overwrite anchor with different schema |
| `NpsError::Identity(msg)` | Key generation, sign/verify, save/load failure |
| `NpsError::Io(msg)` | Network or filesystem error |

All `NpsError` variants implement `std::error::Error` and are compatible with `anyhow` and `thiserror`.

---

## Frame type reference

| Frame | Type code | Crate |
|-------|-----------|-------|
| `AnchorFrame` | 0x01 | `nps-ncp` |
| `DiffFrame` | 0x02 | `nps-ncp` |
| `StreamFrame` | 0x03 | `nps-ncp` |
| `CapsFrame` | 0x04 | `nps-ncp` |
| `HelloFrame` | 0x06 | `nps-ncp` |
| `NopFrame` | 0x07 | `nps-ncp` |
| `ErrorFrame` | 0xFE | `nps-ncp` |
| `QueryFrame` | 0x10 | `nps-nwp` |
| `ActionFrame` | 0x11 | `nps-nwp` |
| `IdentFrame` | 0x20 | `nps-nip` |
| `TrustFrame` | 0x21 | `nps-nip` |
| `RevokeFrame` | 0x22 | `nps-nip` |
| `AnnounceFrame` | 0x30 | `nps-ndp` |
| `ResolveFrame` | 0x31 | `nps-ndp` |
| `GraphFrame` | 0x32 | `nps-ndp` |
| `TaskFrame` | 0x40 | `nps-nop` |
| `DelegateFrame` | 0x41 | `nps-nop` |
| `SyncFrame` | 0x42 | `nps-nop` |
| `AlignStreamFrame` | 0x43 | `nps-nop` |

---

## Building and testing

```bash
# Run all tests
cargo test --workspace

# Build all crates
cargo build --workspace

# Build release
cargo build --workspace --release
```

Test breakdown: `nps-core` 27, `nps-ndp` 25, `nps-nip` 16, `nps-nop` 20. Total: 119.

---

## See also

- [SDK Quickstart](SDK-Quickstart) — language-agnostic first steps and install table
- [SDK Identity and Authentication](SDK-Identity-and-Authentication) — NipIdentity, IdentFrame, trust chain
- [SDK Common Patterns](SDK-Common-Patterns) — anchor cache, streaming, DNS TXT fallback

---

*Last reviewed at suite version: v1.0.0-alpha.14*
