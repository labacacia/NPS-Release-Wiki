# Protocol: NCP — Neural Communication Protocol

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

**Spec**: `spec/NPS-1-NCP.md` v0.6 · **Port**: 17433 (shared, suite-wide)

NCP is the wire-format and transport foundation of the entire NPS suite. Every higher-layer protocol — NWP, NIP, NDP, NOP — is carried as NCP frames. Think of it as HTTP/2 frames plus TCP: NCP defines *how bytes are shaped on the wire* and *how connections are established*, while the upper protocols define what those bytes mean. All NPS traffic arrives on port 17433; the Frame Type byte in each frame's header routes it to the correct protocol handler.

Related: [Protocol NWP](Protocol-NWP) | [Protocol Stack Architecture](Protocol-Stack-Architecture) | [Reference: Frame Registry](Reference-Frame-Registry)

---

## Transport Modes

NCP supports two transport modes. The frame wire format is **identical** in both; only the carrier differs.

| | HTTP Mode | Native Mode |
|---|---|---|
| **Connection** | Standard HTTP/1.1 or HTTP/2 | Direct TCP or QUIC to port 17433 |
| **Framing** | One NCP frame per HTTP request/response body; `Content-Type: application/nwp-frame` | NCP frame bytes directly on the wire |
| **TLS** | Standard HTTPS | Application-managed or raw TCP (dev/test) |
| **Firewall friendliness** | Passes most corporate firewalls unmodified | Requires port 17433 open |
| **Recommended phase** | Phase 1 (recommended default) | Phase 2+ |
| **Connection setup** | `X-NWP-*` request headers carry agent identity | 8-byte preamble + HelloFrame + CapsFrame handshake |

**HTTP mode example:**
```
POST /nwp/products/query HTTP/1.1
Host: api.example.com:17433
Content-Type: application/nwp-frame
X-NWP-Agent: urn:nps:agent:ca.innolotus.com:550e8400

[NCP frame bytes]
```

**Native mode connection sequence:**
```
Client (Agent)                        Server (Node)
      |                                     |
      |-- TCP/QUIC connect ------------->   |
      |-- 8 bytes "NPS/1.0\n" (preamble) -> |  validate; close silently on mismatch
      |-- HelloFrame (0x06) ------------>   |  declare version + capabilities
      |                                     |  version negotiation
      |  <----------- CapsFrame (0x04) ---  |  negotiated session parameters
      |                                     |
      |  [optional] preload AnchorFrames    |
      |-- GET /.schema ---------------->   |
      |  <----------- AnchorFrame(s) -----  |
      |                                     |
      |  -- ESTABLISHED -- ESTABLISHED --   |
```

---

## Frame Structure

Every NCP frame begins with a fixed header, followed immediately by the payload.

### Default Header (EXT=0, payload up to 64 KB)

```
 Byte 0          Byte 1          Bytes 2-3
+--------------+--------------+----------------------------+
|  Frame Type  |    Flags     |      Payload Length        |
|   (1 byte)   |   (1 byte)   |   (2 bytes, big-endian)    |
+--------------+--------------+----------------------------+
|                 Payload (0 - 65,535 bytes)               |
+----------------------------------------------------------+
```

### Extended Header (EXT=1, payload up to 4 GB)

```
 Byte 0          Byte 1          Bytes 2-5           Bytes 6-7
+--------------+--------------+------------------+------------+
|  Frame Type  |    Flags     |  Payload Length  |  Reserved  |
|   (1 byte)   |   (1 byte)   |  (4 bytes, BE)   |  (2 bytes) |
+--------------+--------------+------------------+------------+
|                 Payload (0 - 4,294,967,295 bytes)           |
+-------------------------------------------------------------+
```

The reserved bytes in the extended header MUST be set to 0 by senders and MUST be ignored by receivers.

### Flags Byte (bit-level)

```
Bit 7   Bit 6   Bit 5   Bit 4   Bit 3   Bit 2   Bit 1   Bit 0
+-------+-------+-------+-------+-------+-------+-------+-------+
|  EXT  |  RSV  |  RSV  |  RSV  |  ENC  | FINAL |  T1   |  T0   |
+-------+-------+-------+-------+-------+-------+-------+-------+
```

| Bits | Name | Description |
|------|------|-------------|
| 0–1 | T0, T1 | Encoding tier: `00` = Tier-1 JSON, `01` = Tier-2 MsgPack, `10`/`11` = Reserved |
| 2 | FINAL | Final-chunk flag for StreamFrame; fixed to 1 on all other frame types |
| 3 | ENC | Payload is E2E-encrypted at the application layer (see E2E Encryption below) |
| 4–6 | RSV | Reserved; senders MUST set to 0, receivers MUST ignore |
| 7 | EXT | `0` = default 4-byte header (payload <= 64 KB), `1` = extended 8-byte header (payload <= 4 GB) |

---

## The NPS/1.0\n Preamble (RFC-0001)

> Introduced by NPS-RFC-0001. Mandatory for native mode from NCP v0.6 onward. HTTP mode is unaffected.

In native mode, the very first thing a client sends — before any `HelloFrame` or other NCP frame — is an 8-byte constant preamble:

```
Offset  0   1   2   3   4   5   6   7
       +---+---+---+---+---+---+---+---+
       | N | P | S | / | 1 | . | 0 |\n |
       +---+---+---+---+---+---+---+---+
       0x4E 50  53  2F  31  2E  30  0A
```

The preamble is **not a frame** — it has no header, no flags, no length field. It is raw constant bytes that allow the server to reject misrouted traffic (port scanners, HTTP probes, misconfigured reverse proxies) before any frame parser is exposed, at the cost of a single 8-byte `memcmp`.

**Client behavior:** write all 8 bytes in a single buffer. The client MAY pipeline `HelloFrame` immediately after the preamble without waiting for acknowledgement — server validation is a constant-time byte comparison requiring no round-trip.

**Server behavior and graceful fallback:**
1. On accept, read exactly 8 bytes.
2. If they match `b"NPS/1.0\n"`, proceed to frame parsing.
3. If they do not match: close the connection within 500 ms. **Do not send an `ErrorFrame`** — the peer may not speak NCP, and emitting framed bytes would leak protocol structure to scanners. The server MAY log the first 32 received bytes for operational diagnosis.
4. If fewer than 8 bytes arrive within 10 seconds, close silently (timeout; no error code emitted on the wire).

**Version semantics:** A v1.x server receiving `NPS/2.0\n` MUST close the connection. It MAY write a single 32-byte diagnostic line `NPS-PREAMBLE-UNSUPPORTED-VERSION\n` before closing, since the peer identified itself as an NPS speaker. The minor-version byte (the `0` in `1.0`) is reserved; v1 clients MUST send `0` and v1 servers MUST accept only `0`.

**Server connection state machine:**
```
[LISTEN] --accept--> [PREAMBLE-WAIT] --8-byte match--> [FRAMING] --HelloFrame--> [HANDSHAKE] --CapsFrame--> [ESTABLISHED]
                           |
                           +--mismatch--> [CLOSING] (<= 500 ms)
                           +--10 s timeout--> [CLOSING] (silent)
```

The byte `0x4E` (ASCII `N`) is reserved in the frame-type namespace and MUST NOT be assigned to any NCP frame type, since it is the first byte of every native-mode connection.

**Error code:** `NCP-PREAMBLE-INVALID` (`NPS-PROTO-PREAMBLE-INVALID`) — never emitted on the wire; used in SDK-internal telemetry only to classify the silent close.

---

## EXT Flag: Payload Size Limits

| Header Mode | EXT bit | Length Field | Max Payload |
|-------------|---------|--------------|-------------|
| Default | 0 | 2 bytes | 65,535 bytes (~64 KB) |
| Extended | 1 | 4 bytes | 4,294,967,295 bytes (~4 GB) |

The default 64 KB limit covers the vast majority of production scenarios. Extended mode is used for large embeddings, bulk document transfer, or video frames. The negotiated `max_frame_payload` is agreed during the `HelloFrame`/`CapsFrame` handshake. When a payload would exceed the agreed limit, the sender MUST chunk it using `StreamFrame` sequences.

---

## Frame Types

### AnchorFrame (0x01)

The schema-anchor frame, **published by a Node** to establish a shared schema reference. Agents cache these by `anchor_id` (a SHA-256 digest of the canonical JSON schema computed per RFC 8785 JCS) and reference the cached schema using `anchor_ref` in subsequent requests, eliminating redundant schema transmission. Default TTL is 3600 seconds; `ttl=0` means do not cache.

The Node is always the schema owner; Agents are read-only consumers. When an Agent references a stale `anchor_ref` (the schema has been updated), the Node responds with the new schema embedded inline as `CapsFrame.inline_anchor` — no extra round-trip required. If the `anchor_ref` is completely unknown, the Node returns `NCP-ANCHOR-NOT-FOUND`.

**anchor_id computation:** use an RFC 8785 JCS library. Do not implement JSON canonicalization by hand — cross-language SDK consistency depends on using a standard library.

### DiffFrame (0x02)

Incremental data frame carrying only changed fields, used for subscriptions and polling scenarios. References the base schema via `anchor_ref` and the base version via `base_seq`. Supports two patch formats: `json_patch` (RFC 6902 array of operations, default, Tier-1 compatible) and `binary_bitset` (bitset of changed-field positions followed by new values, ~15–20% more compact, Tier-2 MsgPack only). Using `binary_bitset` in Tier-1 JSON mode is a protocol error.

### StreamFrame (0x03)

Streaming data-chunk frame for large datasets, real-time push, or fragmentation of oversized payloads. Frames in a stream share a `stream_id` (UUID v4) and carry a monotonically increasing `seq` starting at 0. The final chunk sets `is_last=true` (also reflected in the FINAL flag bit). Application-layer flow control uses the `window_size` field: if the receiver emits `window_size=0`, the sender MUST pause until a non-zero update arrives from the receiver.

In HTTP mode, each HTTP request or response body MUST contain exactly one complete NCP frame. Multiple frames MUST NOT be packed into one HTTP body.

### CapsFrame (0x04)

The standard response frame. Encapsulates a full response body with `anchor_ref`, `count`, and `data` array. Also carries optional `token_est` (estimated Cognon consumption, see [Reference: Cognon Budget](Reference-Cognon-Budget)), `cached` flag, `next_cursor` for pagination, and `inline_anchor` for zero-RTT schema updates.

During native-mode connection setup the server returns a negotiation `CapsFrame` (using `anchor_ref: "nps:system:caps"`) containing the agreed `session_version`, `negotiated_encoding`, `max_frame_payload`, `ext_support`, and available `e2e_enc_algorithms`.

### HelloFrame (0x06)

Client-side handshake frame in native mode, sent immediately after the preamble. Declares the client's NPS version range (`nps_version`, `min_version`), supported encodings, supported protocols, optional `agent_id` (NIP NID), `max_frame_payload`, `ext_support`, and `e2e_enc_algorithms`. MUST use Tier-1 JSON (`T0=T1=0`) and MUST NOT set ENC=1 since encoding and encryption are not yet negotiated. The server MUST respond with a `CapsFrame` or `ErrorFrame` within 5 seconds; the client SHOULD disconnect if no response arrives within that window.

HelloFrame is used only in native mode. HTTP mode uses `X-NWP-*` headers to carry the same information.

### ErrorFrame (0xFE)

The **universal error frame** for all NPS protocols. Carries:
- `status`: an NPS status code (transport-level category, e.g. `NPS-CLIENT-NOT-FOUND`)
- `error`: a protocol-level error code (e.g. `NCP-ANCHOR-NOT-FOUND`, `NWP-ACTION-NOT-FOUND`)
- `message`: optional human-readable description
- `details`: optional structured context

Every protocol layer uses the same `ErrorFrame` — there are no separate per-protocol error frame types. In HTTP mode, errors are returned as JSON bodies with `Content-Type: application/nwp-error+json`; the structure mirrors the `ErrorFrame` fields.

---

## Frame Type Namespace

Port 17433 carries all NPS protocols. The Frame Type byte provides routing:

| Range | Protocol |
|-------|----------|
| 0x01–0x0F | NCP |
| 0x10–0x1F | NWP |
| 0x20–0x2F | NIP |
| 0x30–0x3F | NDP |
| 0x40–0x4F | NOP |
| 0x4E | Reserved (preamble first byte — MUST NOT be assigned to any frame type) |
| 0xF0–0xFF | Reserved (0xFE = ErrorFrame) |

---

## Encoding Tiers

| Tier | Format | Flag bits (T1,T0) | Use case |
|------|--------|-------------------|----------|
| Tier-1 | JSON | `00` | Development, debugging, cross-tool compatibility |
| Tier-2 | MsgPack | `01` | Production (~60% size reduction over JSON) |
| — | Reserved | `10` | Future high-performance encoding (not yet allocated) |
| — | Reserved | `11` | Reserved |

Tier-2 MsgPack is the production default and is preferred whenever both sides negotiate it. Tier-1 JSON is mandatory for handshake frames (`HelloFrame`) and is preferred during development. The former Tier-3 MatrixTensor concept was removed; the `10` bit pattern is reserved until a formal RFC allocates it.

---

## E2E Encryption (ENC Flag)

When `ENC=1` in the Flags byte, the payload uses application-layer end-to-end encryption, independent of TLS. This applies when frames pass through untrusted Relay nodes in multi-hop scenarios. The algorithm is negotiated via `e2e_enc_algorithms` in `HelloFrame`/`CapsFrame`.

Supported algorithms: `aes-256-gcm` and `chacha20-poly1305`. Both use a 256-bit key and a 12-byte nonce (generated fresh per frame via CSPRNG). AAD is the 4 or 8-byte frame header.

**Encrypted payload layout:**
```
+----------------+---------------------------+-------------+
|   Nonce        |   Encrypted Payload       |  Auth Tag   |
|  (12 bytes)    |   (ciphertext)            |  (16 bytes) |
+----------------+---------------------------+-------------+
```

The `Payload Length` header field covers all three parts: 12 + len(ciphertext) + 16. Key management (key distribution and rotation) is handled by NIP — NCP defines the framing only. If a frame arrives with ENC=1 but no algorithm was negotiated for the session, the receiver MUST return `NCP-ENC-NOT-NEGOTIATED` and drop the frame.

---

## Common Gotchas

- **EXT=1 header is 8 bytes, not 4.** Code that hard-codes a 4-byte offset will misparse the payload length on extended frames. Always inspect the EXT bit before reading the length field.
- **HTTP mode: one frame per HTTP body.** Each HTTP request/response body MUST contain exactly one complete NCP frame. For chunked data, use StreamFrame sequences delivered via dedicated streaming endpoints.
- **ErrorFrame is universal.** `0xFE` is used by NCP, NWP, NIP, NDP, and NOP. Never introduce a protocol-specific error frame; always use `ErrorFrame` with the appropriate `error` code prefix.
- **anchor_id computation requires JCS.** Use an RFC 8785-compliant library. Hand-rolled JSON canonicalization is a common source of cross-SDK `anchor_id` mismatches.
- **HelloFrame must be Tier-1 JSON with ENC=0.** Encoding and encryption are not yet negotiated when `HelloFrame` is sent.
- **Preamble is native mode only.** HTTP mode clients MUST NOT send the preamble — doing so will corrupt the HTTP framing.

---

## NCP Error Codes

| Error Code | NPS Status | When |
|------------|------------|------|
| `NCP-PREAMBLE-INVALID` | `NPS-PROTO-PREAMBLE-INVALID` | Native-mode preamble mismatch (silent close, no ErrorFrame emitted) |
| `NCP-VERSION-INCOMPATIBLE` | `NPS-PROTO-VERSION-INCOMPATIBLE` | Client `min_version` > server maximum supported version |
| `NCP-ANCHOR-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | Referenced `anchor_ref` not cached at this node |
| `NCP-ANCHOR-STALE` | `NPS-CLIENT-CONFLICT` | Schema exists but has been updated (response includes `inline_anchor`) |
| `NCP-ANCHOR-SCHEMA-INVALID` | `NPS-CLIENT-BAD-FRAME` | AnchorFrame schema is malformed |
| `NCP-ANCHOR-ID-MISMATCH` | `NPS-CLIENT-CONFLICT` | Same `anchor_id` received with a different schema (anchor-poisoning defense) |
| `NCP-FRAME-UNKNOWN-TYPE` | `NPS-CLIENT-BAD-FRAME` | Unknown frame-type byte |
| `NCP-FRAME-PAYLOAD-TOO-LARGE` | `NPS-LIMIT-PAYLOAD` | Payload exceeds the negotiated `max_frame_payload` |
| `NCP-FRAME-FLAGS-INVALID` | `NPS-CLIENT-BAD-FRAME` | Reserved flag bits are non-zero |
| `NCP-STREAM-SEQ-GAP` | `NPS-STREAM-SEQ-GAP` | Non-contiguous `StreamFrame` sequence number |
| `NCP-STREAM-NOT-FOUND` | `NPS-STREAM-NOT-FOUND` | `stream_id` does not refer to an existing stream |
| `NCP-STREAM-LIMIT-EXCEEDED` | `NPS-STREAM-LIMIT` | Maximum concurrent streams per connection exceeded |
| `NCP-STREAM-WINDOW-OVERFLOW` | `NPS-STREAM-LIMIT` | Sender continued after the flow-control window was exhausted |
| `NCP-ENCODING-UNSUPPORTED` | `NPS-SERVER-ENCODING-UNSUPPORTED` | Requested encoding tier not supported |
| `NCP-DIFF-FORMAT-UNSUPPORTED` | `NPS-CLIENT-BAD-FRAME` | `DiffFrame` used `binary_bitset` but receiver does not support it |
| `NCP-ENC-NOT-NEGOTIATED` | `NPS-CLIENT-BAD-FRAME` | ENC=1 frame received but no E2E algorithm was negotiated for this session |
| `NCP-ENC-AUTH-FAILED` | `NPS-CLIENT-BAD-FRAME` | E2E Auth Tag verification failed (possible tampering) |

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
