# Protocol: NOP — Neural Orchestration Protocol

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

**Spec**: `spec/NPS-5-NOP.md` v0.4 · **Port**: 17433 (shared) / 17437 (optional dedicated)
**Supersedes**: NCP AlignFrame (0x05) — deprecated, removed in NPS v1.0

NOP is the orchestration layer of NPS — a combination of SMTP, message queue, and workflow engine at the wire level. It defines how Agents coordinate multi-step tasks: breaking work into DAGs, delegating subtasks to Worker Agents with bounded scope, synchronizing partial results at fan-in barriers, and streaming progress back to the Orchestrator. NOP depends on NCP (framing), NWP (action invocation), and NIP (scope enforcement at every delegation step).

Related: [Protocol NIP](Protocol-NIP) | [Reference: Cognon Budget](Reference-Cognon-Budget) | [SDK Building an Anchor Node](SDK-Building-an-Anchor-Node)

---

## Protocol Overview

The NOP stack position:

```
NOP (Multi-Agent Orchestration)
  +-- NWP (Data & Operation Access)
  |     +-- NCP (Transport Frames & Encoding)
  +-- NIP (Identity & Scope Validation)
```

The Orchestrator calls Worker Agent operations via NWP `ActionFrame`; Worker Agents push intermediate and final results back to the Orchestrator via `AlignStream`; every delegation step is validated by NIP against a carved-down scope.

NOP replaces the deprecated NCP `AlignFrame (0x05)`, which lacked task context binding and NIP identity enforcement. All new code MUST use `AlignStream (0x43)`.

---

## DAG Model: TaskFrame (0x40)

The Orchestrator submits a `TaskFrame` containing a complete Directed Acyclic Graph (DAG) describing the multi-Agent workflow.

### Hard Limits

| Limit | Value | Error on violation |
|-------|-------|-------------------|
| Max DAG nodes | **32** | `NOP-TASK-DAG-TOO-LARGE` |
| Max delegation chain depth | **3 levels** | `NOP-DELEGATE-CHAIN-TOO-DEEP` |
| Overall task timeout | max **3,600,000 ms** (1 hour) | `NOP-TASK-TIMEOUT` |
| CEL condition expression length | max **512 characters** | `NOP-CONDITION-EVAL-ERROR` |
| `input_mapping` JSONPath nesting depth | max **8 levels** | `NOP-INPUT-MAPPING-ERROR` |

### DAG Structure

A DAG consists of `nodes` (vertices) and `edges` (directed edges).

**DAG node fields:**

| Field | Description |
|-------|-------------|
| `id` | Node identifier (unique within DAG) |
| `action` | Operation URL (`nwp://...`) |
| `agent` | NID of the Worker Agent to execute this node |
| `input_from` | Upstream node IDs this node depends on; empty/absent = root node |
| `input_mapping` | JSONPath expressions mapping upstream outputs to this node's params: `{"local_param": "$.upstream_id.result.field"}` |
| `timeout_ms` | Per-node timeout; takes precedence over TaskFrame's global `timeout_ms` |
| `retry_policy` | `{max_retries, backoff: "fixed"/"linear"/"exponential", initial_delay_ms, max_delay_ms, retry_on: [error_codes]}` |
| `condition` | CEL subset expression; if evaluates to `false`, the node is skipped (`SKIPPED` status). If a terminal node is skipped, the task ends `COMPLETED`. |

The DAG MUST have at least one root node (no incoming edges) and at least one terminal node (no outgoing edges). Cycles are rejected with `NOP-TASK-DAG-CYCLE`.

### TaskFrame Top-Level Fields

| Field | Description |
|-------|-------------|
| `task_id` | UUID v4 task identifier |
| `dag` | The DAG object with `nodes` and `edges` arrays |
| `timeout_ms` | Overall task timeout (default 30000, max 3600000) |
| `max_retries` | Global max retries for any single node (default 2) |
| `priority` | `"low"` / `"normal"` / `"high"` |
| `callback_url` | `https://` URL for task completion/failure notification |
| `preflight` | If true, perform a resource pre-flight check before execution |
| `context` | Pass-through context: `{session_id, trace_id, span_id, trace_flags, baggage, custom}` — supports OpenTelemetry W3C TraceContext for distributed tracing across Agent hops |
| `request_id` | UUID v4 for tracing |

---

## DelegateFrame (0x41)

The Orchestrator emits a `DelegateFrame` per DAG node to dispatch a subtask to the designated Worker Agent.

**Key fields:**

| Field | Description |
|-------|-------------|
| `parent_task_id` | Parent `task_id` |
| `subtask_id` | UUID v4 for this subtask |
| `node_id` | Corresponding DAG node `id` |
| `target_agent_nid` | NID of the Worker Agent being delegated to |
| `action` | Operation URL (`nwp://`) |
| `params` | Processed input parameters (after `input_mapping` evaluation) |
| `delegated_scope` | Carved-down NIP scope: `{nodes, actions, max_token_budget}` |
| `deadline_at` | Subtask deadline (ISO 8601 UTC) |
| `idempotency_key` | Idempotency key — MUST be the same value on retries |
| `context` | Inherited from `TaskFrame.context` with `span_id` updated to the current DelegateFrame span |

### Delegation Chain Depth Limit

The maximum delegation chain depth is **3 levels**, counted as entities in the chain:

- Level 1: Orchestrator
- Level 2: Worker Agent (first delegation)
- Level 3: Sub-Worker Agent (second delegation from a Worker)

A fourth entity (Sub-Sub-Worker) would be at level 4, violating the limit. Equivalently, at most 2 `DelegateFrame` hops are permitted. Violations return `NOP-DELEGATE-CHAIN-TOO-DEEP`.

### Scope Enforcement

`delegated_scope.nodes`, `delegated_scope.actions`, and `delegated_scope.max_token_budget` MUST all be subsets of the parent Agent's scope. The CA enforces this at signing time; violations are rejected with `NOP-DELEGATE-SCOPE-VIOLATION`. This is the no-scope-expansion principle inherited from NIP — no Agent in the delegation chain can grant permissions it does not itself possess.

---

## SyncFrame (0x42): K-of-N Fan-In Barrier

`SyncFrame` is a synchronization barrier that waits for dependent subtasks to complete before the DAG proceeds. It supports K-of-N semantics: wait for at least K out of N upstream subtasks to succeed.

**Key fields:**

| Field | Description |
|-------|-------------|
| `task_id` | Parent task |
| `sync_id` | UUID v4 sync point identifier |
| `wait_for` | Array of `subtask_id` values to wait for |
| `min_required` | K: minimum successful subtasks to unblock; 0 or absent = all must succeed |
| `aggregate` | Result aggregation: `"merge"` (default) / `"first"` / `"all"` / `"fastest_k"` |
| `timeout_ms` | Wait timeout; returns `NOP-SYNC-TIMEOUT` on expiry |

When `min_required: K` with K < N, the barrier unblocks as soon as K subtasks succeed. The Orchestrator SHOULD then send cancel signals to the remaining in-flight subtasks.

**Aggregation strategies:**
- `merge`: merge all successful result fields into one object (later keys overwrite)
- `first`: use the first subtask to succeed
- `all`: preserve all results as an array
- `fastest_k`: collect the `min_required` fastest completions in `all` format

---

## AlignStreamFrame (0x43): Streaming Progress

`AlignStream` carries partial results and progress updates from a running subtask back to the Orchestrator. It replaces the deprecated NCP `AlignFrame (0x05)` and adds DAG context binding and NIP identity enforcement.

**Key fields:**

| Field | Description |
|-------|-------------|
| `stream_id` | UUID v4 stream identifier |
| `task_id` | Associated parent `task_id` |
| `subtask_id` | Associated subtask |
| `seq` | Strictly increasing sequence number from 0 |
| `payload_ref` | `anchor_ref` of the payload schema |
| `data` | Intermediate result data |
| `window_size` | Token-level backpressure window in CGN units (not bytes; see CGN Budget Flow below) |
| `is_final` | `true` marks the final result frame for this subtask |
| `sender_nid` | Sender NID — receiver MUST verify this matches the connection identity |
| `error` | Optional error object on final frame: `{code, message, retryable}` |

**Comparison with NCP StreamFrame:**

| | StreamFrame (0x03) | AlignStream (0x43) |
|--|--------------------|--------------------|
| Use case | General data streams (query results) | Multi-Agent task intermediate results |
| Context | None | Carries `task_id` + `subtask_id` |
| Identity binding | None | `sender_nid` mandatory verification |
| Backpressure unit | Bytes / frame count | CGN Token count |
| Error propagation | None | `error` field with task-failure semantics |

---

## Condition Expressions: CEL Subset

The DAG node `condition` field uses a subset of CEL (Common Expression Language):

- Numeric comparisons: `>`, `>=`, `<`, `<=`, `==`, `!=`
- Boolean logic: `&&`, `||`, `!`
- JSONPath access to upstream results: `$.<node_id>.<field>` syntax
- String comparison and `in` operator

When `condition` evaluates to `false`, the node is marked `SKIPPED`. A `SKIPPED` terminal node does not cause overall task failure — the task ends `COMPLETED`. Evaluation errors (syntax error, reference to non-existent field) return `NOP-CONDITION-EVAL-ERROR`.

Expression length is capped at **512 characters**.

---

## Async vs Sync Invocation

NOP subtasks are fundamentally async — each DAG node is dispatched via `DelegateFrame` and the Orchestrator waits for `AlignStream(is_final=true)`. However:

- **Idempotency:** retries MUST use the same `subtask_id` and `idempotency_key`.
- **Timeout handling:** each node has an optional `timeout_ms`; if not set, the global TaskFrame `timeout_ms` applies. Deadline is expressed in `DelegateFrame.deadline_at`. Expiry returns `NOP-DELEGATE-TIMEOUT`.
- **Retry backoff:** `"fixed"` (constant delay), `"linear"` (delay scales with attempt count), `"exponential"` (default; delay doubles each attempt), all capped by `max_delay_ms`.

**Resource pre-flight** (`preflight: true` on `TaskFrame`): before dispatching any subtask, the Orchestrator sends a lightweight `DelegateFrame(action="preflight")` to each Worker Agent. The Agent responds with its availability, available CGN budget, and estimated queue time. If any Agent reports `available: false`, the Orchestrator MUST abort the entire task and return `NOP-RESOURCE-INSUFFICIENT`.

---

## CGN Budget Flow

Cognon (CGN) budget flows through the DAG dispatch chain. When the Orchestrator creates a `TaskFrame`, it sets `context` fields and per-node scopes that include `max_token_budget`. This budget is carved down at each `DelegateFrame` hop (scope carving principle from NIP).

`AlignStream.window_size` provides token-level backpressure in CGN units. The `window_size` value represents the maximum CGN cost the receiver can currently absorb. Before sending each `data` frame, the sender estimates the CGN cost and checks the current window balance. If the estimated cost exceeds the window, the sender pauses until the receiver restores the window by emitting a reverse `AlignStream(data=null, window_size=N)`.

For pre-flight, the Orchestrator provides `estimated_npt` (estimated Cognons) to help Workers self-assess availability.

See [Reference: Cognon Budget](Reference-Cognon-Budget) for CGN computation details.

---

## Task Execution State Machine

```
          +----------+
          | PENDING  |  TaskFrame submitted, awaiting scheduling
          +----+-----+
               |
               v
       +--------------+
       |  PREFLIGHT   |  Resource pre-flight (when preflight=true)
       +------+-------+
              |
              v
          +--------+
          | RUNNING |  Subtasks executing
          +----+----+
               |
      +--------+--------+----------+
      v                 v          v
+------------+     +--------+  +----------+
|WAITING_SYNC|     | FAILED |  |CANCELLED |
|(sync barrier)    +---+----+  +----------+
+------+-----+        |
       |               | Exceeds max_retries
       v               v
  +-----------+
  | COMPLETED |
  +-----------+
```

Subtask states: `PENDING` → `RUNNING` → `COMPLETED` / `FAILED` / `CANCELLED` / `SKIPPED`

**Cancellation:** the Orchestrator may cancel a running task by (a) disconnecting (Worker Agents MUST detect connection close and stop), (b) sending `DelegateFrame(action="cancel")`, or (c) calling `system.task.cancel` via NWP. On receiving a cancel signal, Worker Agents MUST emit `AlignStream(is_final=true, error.code="NOP-TASK-CANCELLED")`.

---

## Hard Limits Summary

| Limit | Value | Error code |
|-------|-------|-----------|
| Max DAG nodes | 32 | `NOP-TASK-DAG-TOO-LARGE` |
| Max delegation chain depth | 3 (Orchestrator + 2 hops) | `NOP-DELEGATE-CHAIN-TOO-DEEP` |
| Max task timeout | 3,600,000 ms (1 hour) | `NOP-TASK-TIMEOUT` |
| Max CEL condition length | 512 characters | `NOP-CONDITION-EVAL-ERROR` |
| Max `input_mapping` JSONPath nesting | 8 levels | `NOP-INPUT-MAPPING-ERROR` |
| `callback_url` scheme | `https://` only; SSRF-validated | `NOP-TASK-DAG-INVALID` |

---

## Error Codes

| Error Code | NPS Status | Description |
|------------|------------|-------------|
| `NOP-TASK-NOT-FOUND` | `NPS-CLIENT-NOT-FOUND` | `task_id` does not exist |
| `NOP-TASK-TIMEOUT` | `NPS-SERVER-TIMEOUT` | Overall task timeout exceeded |
| `NOP-TASK-DAG-INVALID` | `NPS-CLIENT-BAD-FRAME` | DAG format invalid (missing root/terminal, field errors, etc.) |
| `NOP-TASK-DAG-CYCLE` | `NPS-CLIENT-BAD-FRAME` | DAG contains a cycle |
| `NOP-TASK-DAG-TOO-LARGE` | `NPS-CLIENT-BAD-FRAME` | DAG node count exceeds 32 |
| `NOP-TASK-ALREADY-COMPLETED` | `NPS-CLIENT-CONFLICT` | Task already completed; cannot resubmit |
| `NOP-TASK-CANCELLED` | `NPS-CLIENT-CONFLICT` | Task has been cancelled |
| `NOP-DELEGATE-SCOPE-VIOLATION` | `NPS-AUTH-FORBIDDEN` | `delegated_scope` exceeds parent Agent scope |
| `NOP-DELEGATE-REJECTED` | `NPS-CLIENT-UNPROCESSABLE` | Worker Agent rejected delegation (insufficient capability or overloaded) |
| `NOP-DELEGATE-CHAIN-TOO-DEEP` | `NPS-CLIENT-BAD-PARAM` | Delegation chain depth exceeds 3 levels (at most 2 DelegateFrame hops) |
| `NOP-DELEGATE-TIMEOUT` | `NPS-SERVER-TIMEOUT` | Subtask did not complete before `deadline_at` |
| `NOP-SYNC-TIMEOUT` | `NPS-SERVER-TIMEOUT` | SyncFrame barrier timed out waiting for dependent subtasks |
| `NOP-SYNC-DEPENDENCY-FAILED` | `NPS-CLIENT-UNPROCESSABLE` | Failure count exceeded K-of-N tolerance |
| `NOP-STREAM-SEQ-GAP` | `NPS-STREAM-SEQ-GAP` | AlignStream sequence number non-contiguous |
| `NOP-STREAM-NID-MISMATCH` | `NPS-AUTH-UNAUTHENTICATED` | AlignStream `sender_nid` does not match connection identity |
| `NOP-RESOURCE-INSUFFICIENT` | `NPS-SERVER-UNAVAILABLE` | Pre-flight found one or more Worker Agents with insufficient resources (CGN or capability) |
| `NOP-CONDITION-EVAL-ERROR` | `NPS-CLIENT-BAD-PARAM` | DAG node condition expression evaluation failed |
| `NOP-INPUT-MAPPING-ERROR` | `NPS-CLIENT-UNPROCESSABLE` | `input_mapping` JSONPath could not be resolved or target field does not exist |

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
