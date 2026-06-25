# SDK — Go

**Status:** ✅ Content complete — v1.0.0-alpha.13

Go reference implementation of the Neural Protocol Suite. Covers all five sub-protocols: NCP, NWP, NIP, NDP, and NOP.

---

## Installation

```bash
go get github.com/labacacia/NPS-sdk-go@v1.0.0-alpha.13
```

**Requirements:** Go 1.25+.

**Tests:** 106 passing.

**Note:** There is currently no `VERSION` constant exported from the module. If your code needs to check the SDK version at runtime, read it from your own `go.mod`. This is a known gap; a follow-up issue tracks adding a `core.Version` constant.

---

## Package layout

| Package | Protocol | Description |
|---------|----------|-------------|
| `github.com/labacacia/NPS-sdk-go/core` | NCP | Frame types, header codec, `FrameRegistry`, `AnchorFrameCache` |
| `github.com/labacacia/NPS-sdk-go/ncp` | NCP | `AnchorFrame`, `DiffFrame`, `StreamFrame`, `CapsFrame`, `HelloFrame` (`PingIntervalMs`), `NopFrame` (0x07 keepalive), `ErrorFrame` |
| `github.com/labacacia/NPS-sdk-go/nwp` | NWP | `QueryFrame`, `ActionFrame`, `NwpClient` (HTTP mode; reads `X-NWM-Version` and `manifest_version`/`manifest_updated_at` for conditional `.nwm` re-fetch); `ErrAuth*` / `ErrQuery*` / … error code constants |
| `github.com/labacacia/NPS-sdk-go/nip` | NIP | `IdentFrame` (v2 dual-trust; `NodeRoles` self-declared role tags), `TrustFrame`, `RevokeFrame`, `NipIdentity` (Ed25519), `NipIdentVerifier` (RFC-0002 §8.1 dual-trust), `AssuranceLevel` (RFC-0003), `ReputationLogClient` (RFC-0004 Phase 2, added in alpha.7) |
| `github.com/labacacia/NPS-sdk-go/nip/x509` | NIP / RFC-0002 | `IssueLeaf`, `IssueRoot`, `Verify` — NPS X.509 NID certs on stdlib `crypto/x509` (anchored to IANA PEN 65715) |
| `github.com/labacacia/NPS-sdk-go/nip/acme` | NIP / RFC-0002 | `Client` + `Server` (in-process) + JWS/messages — ACME `agent-01` flow |
| `github.com/labacacia/NPS-sdk-go/ndp` | NDP | `AnnounceFrame` (`SpawnSpecRef` structured schema object; `HeartbeatIntervalMs`), `ResolveFrame`, `GraphFrame`, `InMemoryNdpRegistry`, `NdpAnnounceValidator`; DNS TXT fallback (`ResolveViaDns`, `DnsTxtLookup`, `ParseNpsTxtRecord`) |
| `github.com/labacacia/NPS-sdk-go/nop` | NOP | `TaskFrame` (`ResultTtlSeconds`), `DelegateFrame`, `SyncFrame`, `AlignStreamFrame`, `NopClient` |

---

## Key types and functions

### `NpsFrameCodec` — encode an AnchorFrame

```go
import (
    "github.com/labacacia/NPS-sdk-go/core"
    "github.com/labacacia/NPS-sdk-go/ncp"
)

reg   := core.CreateFullRegistry()
codec := core.NewNpsFrameCodec(reg)

frame := &ncp.AnchorFrame{
    AnchorID: "sha256:abc123",
    Schema:   core.FrameDict{"type": "object", "version": "1"},
    TTL:      3600,
}

wire, err := codec.Encode(frame.FrameType(), frame.ToDict(), core.EncodingTierMsgPack, true)
// wire is ready to send over the network

ft, dict, err := codec.Decode(wire)
received := ncp.AnchorFrameFromDict(dict)
```

### `AnchorFrameCache`

```go
cache := core.NewAnchorFrameCache()

schema   := core.FrameDict{"type": "object", "fields": []any{"name", "value"}}
anchorID, err := cache.Set(schema, 3600) // 1-hour TTL

schema, err = cache.GetRequired(anchorID) // returns error if expired
```

### `NwpClient`

```go
import "github.com/labacacia/NPS-sdk-go/nwp"

client := nwp.NewNwpClient("http://node.example.com:17433")

// Query
qf := &nwp.QueryFrame{AnchorRef: "sha256:abc123", Filters: map[string]any{"status": "active"}}
capsFrame, err := client.Query(ctx, qf)

// Stream
frames, err := client.Stream(ctx, qf)
for _, sf := range frames {
    fmt.Println(sf.Payload)
}

// Invoke (sync action)
af := &nwp.ActionFrame{Action: "create", Payload: map[string]any{"name": "item"}}
result, err := client.Invoke(ctx, af)

// Async invoke — check result.Async.TaskID
af.Async = true
result, err = client.Invoke(ctx, af)
fmt.Println(result.Async.TaskID)
```

`NwpClient` uses `context.Context` for cancellation throughout. Pass a timeout context to enforce deadlines.

### `NipIdentity`

```go
import "github.com/labacacia/NPS-sdk-go/nip"

id, err := nip.Generate()
fmt.Println(id.PubKeyString()) // "ed25519:<hex>"

// Sign a frame dict
payload := core.FrameDict{"nid": "urn:nps:node:example.com:agent", "pub_key": id.PubKeyString()}
sig := id.Sign(payload)
ok  := id.Verify(payload, sig)

// Verify with just the public key string (no private key needed)
ok = nip.VerifyWithPubKeyStr(payload, "ed25519:<hex>", sig)

// Save / Load (AES-256-GCM + PBKDF2-SHA256, 600k iterations)
err = id.Save("/path/to/identity.json", "my-passphrase")
loaded, err := nip.Load("/path/to/identity.json", "my-passphrase")
```

### `AssuranceLevel` — empty-string case

The Go SDK uses `if wire == ""` to guard the empty-string case:

```go
import "github.com/labacacia/NPS-sdk-go/nip"

level := nip.AssuranceLevelFromWire("")    // → nip.AssuranceLevelAnonymous
level  = nip.AssuranceLevelFromWire("L1") // → nip.AssuranceLevelL1
```

An empty string returns `Anonymous` rather than an error.

### DNS TXT fallback (`ResolveViaDns`)

```go
import "github.com/labacacia/NPS-sdk-go/ndp"

result := ndp.ResolveViaDns("nwp://example.com/agent", nil)
// result.Host, result.Port, result.Protocol
// pass a DnsTxtLookup implementation to inject a mock in tests
```

### `NopClient` — orchestration

```go
import "github.com/labacacia/NPS-sdk-go/nop"

client := nop.NewNopClient("http://orchestrator.example.com:17433")

tf := &nop.TaskFrame{
    TaskID: "task-" + uuid,
    DAG:    map[string]any{...},
}
taskID, err := client.Submit(ctx, tf)

// Poll status
status, err := client.GetStatus(ctx, taskID)
fmt.Println(status.State()) // "running"

// Wait for completion (polls every 500ms)
status, err = client.Wait(ctx, taskID, nil)
fmt.Println(status.State())        // "completed"
fmt.Println(status.NodeResults())  // map[string]any

err = client.Cancel(ctx, taskID)
```

### Backoff strategies (NOP)

```go
delay := nop.ComputeDelayMs(nop.BackoffExponential, 100, 5000, attempt)
// Also: nop.BackoffFixed, nop.BackoffLinear
```

---

## Idiomatic Go patterns

- All errors are returned as the last return value — no panics in the hot path.
- `context.Context` is the first argument on every network call; use it for cancellation and deadlines.
- Streaming results are returned as slices (`[]StreamFrame`) rather than channels; integrate with your own channels if you need fan-out.
- The `DnsTxtLookup` interface on `ResolveViaDns` lets you inject any DNS backend, including test stubs.

---

## Frame type reference

| Frame | Type code | Package |
|-------|-----------|---------|
| `AnchorFrame` | 0x01 | `ncp` |
| `DiffFrame` | 0x02 | `ncp` |
| `StreamFrame` | 0x03 | `ncp` |
| `CapsFrame` | 0x04 | `ncp` |
| `HelloFrame` | 0x06 | `ncp` |
| `NopFrame` | 0x07 | `ncp` |
| `ErrorFrame` | 0xFE | `ncp` |
| `QueryFrame` | 0x10 | `nwp` |
| `ActionFrame` | 0x11 | `nwp` |
| `IdentFrame` | 0x20 | `nip` |
| `TrustFrame` | 0x21 | `nip` |
| `RevokeFrame` | 0x22 | `nip` |
| `AnnounceFrame` | 0x30 | `ndp` |
| `ResolveFrame` | 0x31 | `ndp` |
| `GraphFrame` | 0x32 | `ndp` |
| `TaskFrame` | 0x40 | `nop` |
| `DelegateFrame` | 0x41 | `nop` |
| `SyncFrame` | 0x42 | `nop` |
| `AlignStreamFrame` | 0x43 | `nop` |

---

## Running tests

```bash
go test ./...
```

---

## See also

- [SDK Quickstart](SDK-Quickstart) — language-agnostic first steps and install table
- [SDK Identity and Authentication](SDK-Identity-and-Authentication) — NipIdentity, IdentFrame, trust chain
- [SDK Common Patterns](SDK-Common-Patterns) — anchor cache, streaming, DNS TXT fallback

---

*Last reviewed at suite version: v1.0.0-alpha.13*
