# Roadmap and Phase Gates

This roadmap separates **product/code delivery** from **host environment construction**. Environment setup does not mean the corresponding application capability is implemented; Mock Runner tests do not mean a real Worker is running.

## Workstream A — Product and code

| Phase | Scope | Exit criteria | Status |
|---|---|---|---|
| A0 — Product/architecture baseline | Freeze product charter, architecture, decisions, roadmap and Agent handoff rules | Docs agree; code baseline and non-goals explicit | In progress |
| A1 — P0 Manager Core | Intake, immutable inputs, per-task workspace, two-slot FIFO, persisted state, Mock ACP/Runner, SQLite events, CAS, incremental Writeup | Regression/full tests, demo, integrity checks and independent review | **Accepted baseline**: P0-REVIEW-FIX-02 |
| A2 — Real ACP/Worker adapter | Replace Mock Runner behind existing contracts; real session creation, prompt/update collection, permissions, stop/close and recovery | Real ACP smoke in isolated test profile; lifecycle/failure tests; no target access required | Not started |
| A3 — Evidence automation | Connect real Worker events/tool results to Collector; provenance, screenshot capture, artifact closure, audit completeness | Reproducible synthetic E2E; tamper/truncation tests; missing-evidence disclosure | Planned |
| A4 — Skills and tool adapters | Core Skill + Web/Misc/Crypto/Reverse/Pwn playbooks; ordinary native tools; selected fixed-version MCP tools | Category routing tests; no duplicated permission/storage logic; adapter security review | Planned |
| A5 — Execution isolation and authorization | WSL2/Docker boundary, per-task mount policy, resource quotas, target allowlist, one-shot grants, fail-closed behavior | Escape/network tests, CPU/memory limits, shutdown and cleanup tests | Planned; depends on Workstream B |
| A6 — Writeup and export closure | Evidence-linked Writeup, human draft/version conflict handling, Flag state, final ZIP and SHA256SUMS | Consistent snapshot; no fabricated steps; hashes verified | Partially inherited from Workbench; Solver integration to verify |
| A7 — Real challenge pilot | One explicitly authorized practice challenge per category, then staged expansion | Human sign-off, evidence audit, no out-of-scope traffic, repeatable result | Not started |

## Workstream B — Host environment

| Phase | Scope | Handoff status |
|---|---|---|
| B0 — Windows baseline audit | Host capability and constraints inventory | Reported complete |
| B1 — WSL2 networking foundation | WSL2 mirrored networking and proxy path | Reported complete in Phase 2B-1F |
| B2 — Docker daemon proxy inheritance | Ensure dockerd/containerd inherit required proxy configuration | **Known outstanding issue** in 2B-1F handoff |
| B3 — Docker Engine validation | Official packages, daemon startup, image pull and hello-world | Not confirmed complete |
| B4 — CTF toolchain image | Version-pinned tools, non-root runtime, minimal capabilities, resource limits | Not started |
| B5 — Isolation acceptance | Mount, process, network, privilege, resource and cleanup tests | Not started |

Environment changes affecting OS features, reboot, administrator privileges, security controls, firewall, proxy, Docker daemon or persistent host configuration require explicit user approval.

## Immediate order

1. Complete A0 documentation baseline and import the exact accepted P0 source without rewriting behavior.
2. Preserve P0 as a recorded baseline; run tests from this new repository and compare with supplied evidence.
3. Write the A2 real ACP adapter task specification against frozen Runner/ACP contracts.
4. In parallel only when authorized, resolve Workstream B2 (dockerd proxy inheritance) and continue Docker validation.
5. Do not start A3/A4/A5 until relevant contracts and security gates are reviewed.

## Definition of done

Every task needs scope/non-goals, exact files/interfaces changed, actual test commands/output, security/failure-path analysis, evidence/checksums where relevant, updated status/decision log, and reviewer disposition (PASS / PASS-WITH-NOTES / REQUEST-CHANGES / BLOCKED).
