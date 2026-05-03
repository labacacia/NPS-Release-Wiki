# Contributing Guide

> **Audience:** All contributors (PRs, docs, issues, security reports)
> **Status:** STUB — to be authored by nps-main session
> **Source-of-truth precedence:** `spec/` documents in [`labacacia/NPS-Release`](https://github.com/labacacia/NPS-Release/tree/main/spec) win over this page if they disagree.

## Scope

How to contribute to the NPS suite: code style, PR convention, issue routing rule, security disclosure.

## What this page should contain

- Where to file issues: ALL audit / drift / cross-repo findings → `labacacia/NPS-Dev` (project rule, durable). Never file in satellite repos.
- Label taxonomy: `severity:{critical,major,minor}`, `area:{spec,sdk,ingress,daemon,docs,ci}`, GitHub defaults (`bug`, `enhancement`, `documentation`)
- PR convention: `Closes labacacia/NPS-Dev#<n>` for any fix-issue PR; cross-repo PRs reference the same issue
- Code style per language (link to per-impl style docs)
- Documentation: EN primary + CN secondary (`name.cn.md`); language switcher at top of every doc; bilingual parity is a release blocker
- Security: how to report a vulnerability privately
- The "work discipline" rules from past audit prompts: post grep evidence on issue close, no over-claiming commit messages

## Source material to draw from

- `NPS-Release/CONTRIBUTING.md`
- `NPS-Dev/CONTRIBUTING.md`

## Cross-links

- [Release Process](Release-Process)
- [Repository Topology](Repository-Topology)

## TODO checklist

- [ ] Write the introduction (2–3 paragraphs, set context)
- [ ] Add code examples / wire diagrams as appropriate
- [ ] Cross-check field names match current naming (`node_roles` not `node_kind`; `cgn_est` not `estimated_npt`)
- [ ] Verify all referenced spec section numbers against latest spec versions
- [ ] Add a "Last reviewed at suite version: vX.Y.Z" footer once content is written
- [ ] EN content first; CN translation may follow as `Page-Name.cn` if the user requests bilingual wiki
