# Reference: Frame Registry

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

Every NPS frame type is identified by a single byte. Frames from all five protocols share one unified byte space, routed by type code. The machine-readable source of truth is `spec/frame-registry.yaml` in the repository — that file is CI-validated on every commit.

This page provides the human-readable reference for the same data.

## How the Byte Space Works

All NPS connections — native mode and HTTP mode — carry frames prefixed by a 1-byte type identifier. Because all protocols share port 17433 (the unified NPS port), the type byte is the primary routing signal: a node reads the first byte and dispatches the frame to the correct protocol handler.

Each protocol owns a contiguous range:

| Range | Owner |
|-------|-------|
| `0x01–0x0F` | NCP — Neural Communication Protocol |
| `0x10–0x1F` | NWP — Neural Web Protocol |
| `0x20–0x2F` | NIP — Neural Identity Protocol |
| `0x30–0x3F` | NDP — Neural Discovery Protocol |
| `0x40–0x4F` | NOP — Neural Orchestration Protocol |
| `0xF0–0xFF` | System / Reserved |

Bytes not listed in the registry as assigned are **reserved**. Implementations MUST NOT define or handle frames with unassigned bytes unless an RFC explicitly claims the range.

---

## NCP Frames (0x01–0x0F)

Neural Communication Protocol — wire format, framing, streaming, and handshake.

| Byte | Frame Name | Status | Spec Reference | Description |
|------|------------|--------|----------------|-------------|
| `0x01` | AnchorFrame | Stable | NPS-1-NCP §4.1 | Schema anchor published by a Node. `anchor_id` is computed via RFC 8785 JCS + SHA-256. Establishes a global schema reference to eliminate repeated schema transmission. Agents read and cache; they do not author AnchorFrames. |
| `0x02` | DiffFrame | Stable | NPS-1-NCP §4.2 | Incremental data frame. Supports `json_patch` (RFC 6902, Tier-1 default) and `binary_bitset` (Tier-2, ~15–20% smaller) via the `patch_format` field. NPS-CR-0002 extends `event_type` with topology-stream values: `member_joined`, `member_left`, `member_updated`. |
| `0x03` | StreamFrame | Stable | NPS-1-NCP §4.3 | Streaming data chunk with formal flow control (`window_size` backpressure protocol). Used for large datasets, real-time push, or payload fragmentation. |
| `0x04` | CapsFrame | Stable | NPS-1-NCP §4.4 | Capsule response frame. Wraps a complete response body with anchor reference and pagination cursor. Also used for server-side connection capability negotiation. Supports `inline_anchor` for zero-RTT schema refresh. |
| `0x05` | ~~AlignFrame~~ | **Deprecated** | NPS-1-NCP §4.5 | Multi-AI state alignment. **Superseded by NOP AlignStream (0x43).** New implementations MUST NOT emit this frame type. |
| `0x06` | HelloFrame | Stable | NPS-1-NCP §4.6 | Native-mode client handshake frame. Sent by the Agent immediately after TCP/QUIC connect to declare protocol version, encoding capabilities, and `max_frame_payload`. Server responds with CapsFrame. Not used in HTTP mode. |

**Reserved in NCP range:** `0x07–0x0F`.

> **Special reservation — `0x4E`**: The byte `0x4E` (ASCII `N`) is reserved by NPS-RFC-0001 as the first byte of the native-mode connection preamble `b"NPS/1.0\n"`. Servers MUST NOT interpret `0x4E` as a frame type when it is read as the very first byte after the transport handshake.

---

## NWP Frames (0x10–0x1F)

Neural Web Protocol — query, action, and subscription.

| Byte | Frame Name | Status | Spec Reference | Description |
|------|------------|--------|----------------|-------------|
| `0x10` | QueryFrame | Draft | NPS-2-NWP §6 | Structured data query for Memory Nodes. Supports filtering, field selection, cursor pagination, ordering, vector search, streaming mode, aggregation, and (NPS-CR-0002) optional `type` field for reserved namespaces (`topology.snapshot`, `topology.stream`). Unrecognized `type` values MUST be rejected with `NWP-RESERVED-TYPE-UNSUPPORTED`. |
| `0x11` | ActionFrame | Draft | NPS-2-NWP §7 | Operation invocation for Action and Complex Nodes. Supports sync/async execution, idempotency, `callback_url`, priority, `request_id`, and system-reserved actions (`system.task.status`, `system.task.cancel`). |
| `0x12` | SubscribeFrame | Draft | NPS-2-NWP §8 | Change subscription management for Memory Nodes. Actions: `subscribe` / `unsubscribe` / `ping`. Server pushes DiffFrame (0x02) updates with monotonic `seq`. HTTP mode uses Server-Sent Events. Supports optional `filter`, `heartbeat_interval`, and `resume_from_seq` for lossless reconnection. NPS-CR-0002 adds optional `type` field for `topology.stream` subscriptions. |

**Reserved in NWP range:** `0x13–0x1F`.

---

## NIP Frames (0x20–0x2F)

Neural Identity Protocol — identity declarations, trust chains, and revocation.

| Byte | Frame Name | Status | Spec Reference | Description |
|------|------------|--------|----------------|-------------|
| `0x20` | IdentFrame | Draft | NPS-3-NIP §5.1 | Agent identity declaration and certificate. Carries NID, public key, capabilities, scope, optional metadata (`model_family`, `tokenizer`), and (NPS-RFC-0003) optional `assurance_level` (`anonymous` / `attested` / `verified`). Used as the connection handshake. |
| `0x21` | TrustFrame | Draft | NPS-3-NIP §5.2 | Cross-CA trust chain delegation. Allows one organization CA to grant scoped trust to another CA. |
| `0x22` | RevokeFrame | Draft | NPS-3-NIP §5.3 | NID or capability revocation. Takes immediate effect upon CA acknowledgment. |

**Reserved in NIP range:** `0x23–0x2F`.

---

## NDP Frames (0x30–0x3F)

Neural Discovery Protocol — presence announcement, address resolution, and graph synchronization.

| Byte | Frame Name | Status | Spec Reference | Description |
|------|------------|--------|----------------|-------------|
| `0x30` | AnnounceFrame | Draft | NPS-4-NDP §3.1 | Node or Agent presence broadcast. Announces NID, physical addresses, capabilities, TTL, `activation_mode` (`ephemeral` / `resident` / `hybrid`), and (NPS-CR-0001) `node_roles` array (`memory` / `action` / `complex` / `anchor` / `bridge`). Legacy `"gateway"` value in `node_roles` is rejected with `NDP-ANNOUNCE-ROLE-REMOVED`. Must be signed with the IdentFrame private key. |
| `0x31` | ResolveFrame | Draft | NPS-4-NDP §3.2 | Resolves a `nwp://` URL to a physical endpoint (host:port + certificate fingerprint). |
| `0x32` | GraphFrame | Draft | NPS-4-NDP §3.3 | Node graph synchronization and change subscription. Supports full and incremental sync modes. |

**Reserved in NDP range:** `0x33–0x3F`.

---

## NOP Frames (0x40–0x4F)

Neural Orchestration Protocol — DAG task dispatch, delegation, synchronization, and directed streaming.

| Byte | Frame Name | Status | Spec Reference | Description |
|------|------------|--------|----------------|-------------|
| `0x40` | TaskFrame | Draft | NPS-5-NOP §3.1 | Task definition and dispatch. Defines a DAG of subtasks with per-node `timeout`, `retry_policy`, `condition`, and `input_mapping`. Supports preflight resource check, OpenTelemetry context propagation, `callback_url`, and `priority`. Maximum DAG node count: 32. |
| `0x41` | DelegateFrame | Draft | NPS-5-NOP §3.2 | Subtask delegation to a Worker Agent. Carries a carved-down scope (MUST NOT exceed parent scope). Also used for preflight probe (`action="preflight"`) and cancellation (`action="cancel"`). Maximum delegation chain depth: 3 levels. |
| `0x42` | SyncFrame | Draft | NPS-5-NOP §3.3 | Multi-Agent synchronization barrier. Supports K-of-N semantics (`min_required`), aggregation strategies (`merge` / `first` / `all` / `fastest_k`), and per-barrier timeout. |
| `0x43` | AlignStream | Draft | NPS-5-NOP §3.4 | Directed task stream. Supersedes deprecated AlignFrame (0x05). Extends NCP streaming with task DAG context (`task_id` + `subtask_id`), NID binding, CGN-unit backpressure (`window_size`), and error propagation. |

**Reserved in NOP range:** `0x44–0x4F`.

---

## System / Reserved (0xF0–0xFF)

| Byte | Frame Name | Status | Spec Reference | Description |
|------|------------|--------|----------------|-------------|
| `0xFE` | ErrorFrame | Stable | NPS-1-NCP §4.7 | Unified error frame for all protocol layers. Carries NPS status code, protocol error code, and optional diagnostic details. Used in native transport mode. Allocated from the Reserved range and shared across all five protocols. |

**Reserved:** `0xF0–0xFD` and `0xFF`. Implementations MUST NOT define frames in this range without a spec update.

---

## Reserved Ranges Summary

| Range | Status |
|-------|--------|
| `0x05` | Deprecated (AlignFrame, superseded by 0x43) |
| `0x07–0x0F` | Reserved — NCP expansion |
| `0x13–0x1F` | Reserved — NWP expansion |
| `0x23–0x2F` | Reserved — NIP expansion |
| `0x33–0x3F` | Reserved — NDP expansion |
| `0x44–0x4F` | Reserved — NOP expansion |
| `0x4E` | Hard-reserved — native-mode preamble byte (NPS-RFC-0001) |
| `0x50–0xEF` | Unassigned — available for future protocol layers via RFC |
| `0xF0–0xFD`, `0xFF` | System reserved |

---

## How to Claim a New Frame Type

New frame types require a formal RFC. The process is:

1. **Draft an RFC** using the template at `spec/rfcs/template.md`. The RFC must justify the new frame, specify its byte assignment from an appropriate reserved range, and define the full field schema.
2. **Open a PR** against the `dev` branch following the RFC process described in `spec/rfcs/README.md`. Breaking changes to existing frames require shepherd approval and a minimum discussion window.
3. **After the RFC is accepted**, add the frame entry to `spec/frame-registry.yaml`. CI validates that the YAML is well-formed and that no byte is assigned twice.
4. **Update the owning protocol spec** (`spec/NPS-{n}-*.md`) with the complete field definition table.
5. **Update `spec/NPS-0-Overview.md`** frame namespace table.

Do not implement or ship a new frame type before the RFC is accepted. The `frame-registry.yaml` is the authoritative registry — any byte not listed there as `assigned` is treated as unknown by all conformant implementations.

---

## See Also

- [Protocol NCP](Protocol-NCP) — NCP frame semantics in full

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
