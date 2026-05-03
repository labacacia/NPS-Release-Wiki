# Daemon: nps-runner

> **Audience:** Operators
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

`nps-runner` — task executor daemon. Walks DAG steps dispatched via NOP.

## What this page should contain

- Purpose
- Source: `NPS-Dev/tools/daemons/nps-runner/`
- Distribution: bundled into `labacacia/nps-daemons`
- Required env vars
- /health shape
- Concurrency model + scaling
- Relationship to npsd (typically 1 npsd : N runner)
- NOP DAG execution semantics: how it handles delegate, condition, async actions

## Source material to draw from

- `NPS-Dev/tools/daemons/nps-runner/`
- `spec/NPS-5-NOP.md` for the orchestration semantics it implements

## Cross-links

- [Daemon NPSd](Daemon-NPSd)
- [Protocol NOP](Protocol-NOP)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
