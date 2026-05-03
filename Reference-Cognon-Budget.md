# Reference: Cognon (CGN) Budget

**Status:** ✅ Content complete — v1.0.0-alpha.5.2

## What Is a Cognon?

**CGN (Cognon)** is NPS's standardized unit for measuring AI compute cost. It provides a model-neutral accounting token that lets agents declare spending caps and lets nodes report actual consumption — regardless of which LLM is running at either end.

The name "Cognon" derives from "cognitive unit": a minimal quantum of AI reasoning work. This distinguishes it clearly from overloaded terms like "token" (used for npm tokens, network protocol tokens, authentication tokens, and raw LLM tokens simultaneously).

### Why Not Use Raw LLM Tokens?

Different models count tokens differently. GPT-4o, Claude, Gemini, and LLaMA 3 all use different tokenizers. A response that costs 200 tokens from one model may cost 210 from another. Raw token counts are therefore not portable across a heterogeneous NPS deployment where nodes may run different models.

CGN solves this by defining a reference baseline (GPT-4 / `cl100k_base` = 1.0 CGN per native token) and publishing exchange rates for other model families. Budget caps set in CGN are semantically consistent regardless of which model a node uses internally.

---

## Naming History: NPT → CGN

In NPS versions prior to v1.0.0-alpha.5.2, the same concept was called **NPT** (NPS Protocol Token). NPT was renamed to **CGN (Cognon)** to eliminate confusion with:

- npm authentication tokens (also frequently called "NPT" in toolchain docs)
- Network protocol tokens used in routing and flow control
- The ambiguous phrase "NPS token" used informally in the community

### Wire-Level Breaking Change

The rename is **breaking at the wire level**. The field that carried estimated token cost was renamed:

| Version | Field name |
|---------|------------|
| Prior to alpha.5.2 | `estimated_npt` |
| alpha.5.2 and later | `cgn_est` |

No compatibility alias is retained. Clients using `estimated_npt` will silently fail when communicating with an alpha.5.2 server — the field will simply be absent from the response. Update all SDK code and stored frame parsers before upgrading to alpha.5.2.

---

## Where CGN Appears on the Wire

CGN values appear in multiple places across the NPS protocol stack:

| Location | Field / Header | Description |
|----------|----------------|-------------|
| NWP request | `X-NWP-Budget` header | Maximum CGN the agent is willing to spend on this request (uint32) |
| NWP request | `X-NWP-Tokenizer` header | Tokenizer the agent uses, for accurate CGN calculation on the node side |
| NWP response | `X-NWP-Tokens` header | Actual CGN consumed by this response |
| NWP response | `X-NWP-Tokens-Native` header | Native token count (when the tokenizer is known) |
| NWP response | `X-NWP-Tokenizer-Used` header | Tokenizer identifier actually used by the node |
| NWP ActionSpec (NWM) | `cgn_est` field | Estimated CGN cost of calling an action, as declared in the node manifest |
| NOP DAG node | per-node budget | CGN budget allocated to each subtask node in a TaskFrame |
| CapsFrame | `token_est` field | CGN estimate for the response payload |

---

## Tokenizer Resolution Chain

When an agent makes a request, the node resolves which tokenizer to use for counting CGN in this order:

```
1. Explicit declaration by the agent (X-NWP-Tokenizer header)
   ↓ not declared
2. Auto-match from IdentFrame metadata
   (IdentFrame.metadata.model_family or IdentFrame.metadata.tokenizer)
   ↓ match failed or IdentFrame absent
3. Default fallback: CGN = ceil(UTF-8_bytes / 4)
```

**Priority 1 — Explicit declaration** (highest): the agent sets `X-NWP-Tokenizer: cl100k_base` in the request header. The node MUST use the declared tokenizer. If the node does not support that tokenizer, it SHOULD fall back to auto-match rather than reject the request.

**Priority 2 — Auto-match from IdentFrame**: the node reads `IdentFrame.metadata.model_family` (e.g. `"openai/gpt-4o"`, `"anthropic/claude-4"`) or `IdentFrame.metadata.tokenizer` and selects the matching exchange rate from its built-in table.

**Priority 3 — Default fallback**: when no tokenizer can be determined, the node uses `ceil(UTF-8_bytes / 4)`. This formula reflects the average behavior of mainstream LLM tokenizers (~4 bytes per token for English, ~3 bytes for Chinese) and is the most conservative baseline.

---

## CGN Exchange Rate Table

Nodes SHOULD ship with built-in exchange rates for common model families. The table maps native tokens to CGN:

| Model family | Tokenizer | 1 native token ≈ CGN | Notes |
|---|---|---|---|
| OpenAI GPT-4 / GPT-4o | `cl100k_base` | 1.00 | Reference baseline |
| Anthropic Claude | Claude tokenizer | 1.05 | Slightly higher than GPT-4 |
| Google Gemini | SentencePiece | 0.95 | Slightly lower than GPT-4 |
| Meta LLaMA 3 | `llama3-tokenizer` | 1.02 | Near baseline |
| Mistral | SentencePiece | 0.98 | Near baseline |
| Default (unknown model) | UTF-8 / 4 | 1.00 | Fallback |

The table is maintained with NPS version updates. Node implementations MAY override the built-in rates via hot-reloadable configuration. CGN values are always uint32 (maximum 4,294,967,295).

---

## Request and Response Flow

### Setting a Budget

An agent declares its spending cap in the request header:

```
X-NWP-Budget: 500
X-NWP-Tokenizer: cl100k_base
```

### Over-Budget Handling

When a response would exceed `X-NWP-Budget`:

1. The node SHOULD trim the response first (fewer fields or records) to fit within budget.
2. If trimming is impossible (a single record already exceeds budget), the node MUST return a `NWP-BUDGET-EXCEEDED` error (`NPS-LIMIT-BUDGET`, HTTP 429).
3. The node MUST NOT silently truncate structured data — truncation produces incomplete structures on the agent side.

### Reading Consumption

The node reports actual consumption in response headers:

```
X-NWP-Tokens: 312
X-NWP-Tokens-Native: 298
X-NWP-Tokenizer-Used: cl100k_base
```

---

## Streaming and Subscription Budget Policy

The `X-NWP-Budget` cap applies to **synchronous request/response operations** (QueryFrame → CapsFrame or StreamFrame batch). Continuous-push operations follow modified rules.

### Streaming Queries (QueryFrame `stream: true`)

- `X-NWP-Budget` applies **per StreamFrame batch**, not to the total stream.
- The node MUST trim or stop a batch if processing it would exceed the declared budget.
- `X-NWP-Tokens` in the response header reports the CGN consumed by the current batch only.
- The agent MAY disconnect early once its cumulative budget is exhausted.

### Push Streams (SubscribeFrame / topology.stream / event subscriptions)

Long-running push streams represent an ongoing series of events with no fixed response size. Budget enforcement is therefore **agent-side**:

| Aspect | Behavior |
|--------|----------|
| `X-NWP-Budget` enforcement | Not applied by the node; push events are generated independently of any per-request budget cap |
| `X-NWP-Tokens` reporting | The node SHOULD include this header on each push event (DiffFrame), reporting the CGN for that event's payload |
| Agent-side enforcement | The agent is responsible for tracking cumulative CGN across events and disconnecting when its session budget is exhausted |

Enforcing `X-NWP-Budget` on push streams would require the node to buffer future events, which is incompatible with real-time topology change delivery. Agent-side enforcement is the correct locus for subscription-stream budget control.

---

## Why Use Cognon Instead of Raw Tokens?

**Compute equivalence across models**: a budget of 500 CGN means the same workload cost regardless of whether the responding node runs GPT-4, Claude, or LLaMA 3. Without CGN normalization, agents would need to negotiate separate budgets per-model or per-tokenizer.

**Budget portability**: an orchestrating agent distributing subtasks across a NOP DAG can allocate CGN budgets to subtask nodes without knowing which LLM each Worker Agent uses. The exchange rate table handles the per-model conversion transparently.

**Auditability**: `X-NWP-Tokens` and `X-NWP-Tokens-Native` are both reported, so operators can track both the normalized (CGN) and raw consumption. This is useful for reconciliation with model provider billing.

---

## Implementation Notes

- Node implementations SHOULD ship with at least the `cl100k_base` (GPT-4 family) tokenizer built in.
- The exchange-rate table SHOULD be hot-reloadable configuration, not hard-coded.
- For high-frequency scenarios, token estimation MAY be sampled rather than computed record-by-record.
- The `token_est` field in CapsFrame and the `cgn_est` field in NWM ActionSpec are always in CGN units.

---

## See Also

- [Protocol NWP](Protocol-NWP) — NWP query, action, and subscription framing; `X-NWP-Budget` header semantics
- [Protocol NOP](Protocol-NOP) — NOP DAG per-node budget allocation and AlignStream CGN backpressure

---

*Last reviewed at suite version: v1.0.0-alpha.5.2*
