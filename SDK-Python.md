# SDK — Python

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

Python client library for the Neural Protocol Suite. Covers all five protocols: NCP, NWP, NIP, NDP, and NOP.

---

## Installation

```bash
pip install nps-lib==1.0.0a5.2
```

For development extras (pytest, coverage, linting):

```bash
pip install "nps-lib[dev]==1.0.0a5.2"
```

> **Package name:** The PyPI distribution is `nps-lib`. The name `nps-sdk` is taken by an unrelated package (Ingenico). The Python import namespace is always `nps_sdk`.

**Requirements:** Python 3.11+. Dependencies: `msgpack`, `httpx`, `cryptography`.

**Tests:** 221 passing, ≥ 90% coverage target.

---

## Module layout

| Module | Description |
|--------|-------------|
| `nps_sdk.core` | Frame header, `NpsFrameCodec` (Tier-1 JSON / Tier-2 MsgPack), `FrameRegistry`, `AnchorFrameCache`, exceptions |
| `nps_sdk.ncp` | NCP frames: `AnchorFrame`, `DiffFrame`, `StreamFrame`, `CapsFrame`, `HelloFrame`, `ErrorFrame` |
| `nps_sdk.nwp` | NWP frames: `QueryFrame`, `ActionFrame`; async `NwpClient`; `NwpErrorCodes` (30 constants) |
| `nps_sdk.nip` | NIP frames: `IdentFrame` (v2 dual-trust), `TrustFrame`, `RevokeFrame`; `NipIdentity` (Ed25519); `NipIdentVerifier` + `NipVerifierOptions` (RFC-0002 §8.1 dual-trust); `AssuranceLevel` (RFC-0003) |
| `nps_sdk.nip.x509` | RFC-0002 X.509 NID certs: `NipX509Builder`, `NipX509Verifier`, `NpsX509Oids` |
| `nps_sdk.nip.acme` | RFC-0002 ACME `agent-01`: `AcmeClient`, `AcmeServer` (in-process), JWS helpers, messages |
| `nps_sdk.ndp` | NDP frames: `AnnounceFrame`, `ResolveFrame`, `GraphFrame`; in-memory registry and validator; `resolve_via_dns` DNS TXT fallback |
| `nps_sdk.ndp.dns_txt` | `resolve_via_dns(target, dns_lookup=None)` — looks up `_nps-node.<host>` TXT records when a target is not in the in-memory registry |
| `nps_sdk.nop` | NOP frames: `TaskFrame`, `DelegateFrame`, `SyncFrame`, `AlignStreamFrame`; async `NopClient` |

---

## Type hints

The entire public API is fully annotated. A `py.typed` marker file is included so that mypy and pyright resolve types without extra configuration.

---

## Key classes

### `NpsFrameCodec`

Encodes and decodes NPS frames. Defaults to Tier-2 MsgPack.

```python
from nps_sdk.core.codec import NpsFrameCodec
from nps_sdk.core.registry import FrameRegistry

registry = FrameRegistry.create_default()
codec    = NpsFrameCodec(registry)

wire   = codec.encode(frame)   # bytes — MsgPack by default
result = codec.decode(wire)    # NpsFrame subclass
```

To encode as JSON (debugging only):

```python
from nps_sdk.core.codec import EncodingTier

wire_json = codec.encode(frame, tier=EncodingTier.JSON)
```

### `AnchorFrame`

Content-addressed schema descriptor. The `anchor_id` is a `sha256:<hex>` string computed over the canonical JSON representation of the schema.

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
frame  = AnchorFrame(anchor_id="sha256:abc123", schema=schema, ttl=3600)

wire   = codec.encode(frame)   # bytes — Tier-2 MsgPack by default
result = codec.decode(wire)    # → AnchorFrame
```

### `AnchorFrameCache`

Store and retrieve schemas by anchor ID with TTL eviction:

```python
from nps_sdk.core.cache import AnchorFrameCache

cache     = AnchorFrameCache()
anchor_id = cache.set(frame)               # returns canonical sha256 anchor_id
frame     = cache.get_required(anchor_id)  # raises if missing or expired
```

### `NwpClient` (async)

```python
import asyncio
from nps_sdk.nwp import NwpClient, QueryFrame

async def main():
    async with NwpClient("https://node.example.com") as client:
        caps = await client.query(QueryFrame(anchor_ref="sha256:abc123", limit=50))
        print(caps.count, caps.data)

asyncio.run(main())
```

Streaming:

```python
async with NwpClient("https://node.example.com") as client:
    async for chunk in client.stream(QueryFrame(anchor_ref="sha256:abc123")):
        print(chunk.seq, chunk.data)
```

Action invocation:

```python
from nps_sdk.nwp import ActionFrame

async with NwpClient("https://node.example.com") as client:
    result = await client.invoke(
        ActionFrame(action_id="orders.create", params={"sku": "X-101", "qty": 1})
    )
```

A synchronous fallback is available by wrapping coroutines with `asyncio.run()`.

### `NipIdentity`

Ed25519 identity management:

```python
from nps_sdk.nip.identity import NipIdentity

# Generate and save an encrypted keypair (AES-256-GCM + PBKDF2)
identity = NipIdentity.generate("ca.key", passphrase="my-secret")

# Load from file
identity = NipIdentity()
identity.load("ca.key", passphrase="my-secret")

# Sign a NIP frame payload (canonical JSON, no 'signature' field)
sig = identity.sign(ident_frame.unsigned_dict())

# Verify
ok = NipIdentity.verify_signature(identity.pub_key_string, payload, sig)
```

### `AssuranceLevel` — empty-string fix (alpha.5)

`AssuranceLevel.from_wire("")` returns `ANONYMOUS` instead of raising `ValueError`. This is a spec §5.1.1 fix; the check is `if not wire: return AssuranceLevel.ANONYMOUS`. Prior to alpha.5, an empty string raised `ValueError`, breaking any code that handled nodes which omit the assurance level field entirely.

```python
from nps_sdk.nip.identity import AssuranceLevel

level = AssuranceLevel.from_wire("")    # → AssuranceLevel.ANONYMOUS  (not ValueError)
level = AssuranceLevel.from_wire("L1") # → AssuranceLevel.L1
```

### DNS TXT fallback resolution

When a target is not in the in-memory NDP registry, `resolve_via_dns` looks up `_nps-node.<host>` TXT records in the format `v=nps1 nid=... port=... fp=...`:

```python
from nps_sdk.ndp.dns_txt import resolve_via_dns

result = resolve_via_dns("nwp://example.com/data/items")
# inject a custom resolver callable for testing:
# result = resolve_via_dns("nwp://example.com/data/items", dns_lookup=my_mock_fn)
```

---

## NWP HTTP Overlay paths

`NwpClient` communicates via HTTP with `Content-Type: application/x-nps-frame`.

| Operation | Path | Request Frame | Response Frame |
|-----------|------|---------------|----------------|
| Schema anchor | `POST /anchor` | `AnchorFrame` | 204 |
| Structured query | `POST /query` | `QueryFrame` | `CapsFrame` |
| Streaming query | `POST /stream` | `QueryFrame` | `StreamFrame` chunks |
| Action invocation | `POST /invoke` | `ActionFrame` | raw result or `AsyncActionResponse` |

---

## Running tests

```bash
pytest                 # all tests + coverage report
pytest -k test_nip     # NIP tests only
```

---

## See also

- [SDK Quickstart](SDK-Quickstart) — language-agnostic first steps and install table
- [SDK Identity and Authentication](SDK-Identity-and-Authentication) — NipIdentity, IdentFrame, trust chain
- [SDK Common Patterns](SDK-Common-Patterns) — anchor cache, streaming, DNS TXT fallback

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
