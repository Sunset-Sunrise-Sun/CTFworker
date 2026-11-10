# Roadmap and Phase Gates

Three tracks remain separate:
- **Product P0–P3:** user-facing outcomes and acceptance gates.
- **Engineering A0–A7:** code migration and integration.
- **Environment B0–B5:** WSL2/Docker/toolchain and OS isolation.

PRODUCT_BLUEPRINT.md is authoritative. A phase is complete only when its exit gate is evidenced; a code milestone does not imply a product phase is complete.

## 1. Product roadmap

| Phase | Goal | Deliverables | Exit gate | State |
|---|---|---|---|---|
| P0 — Prerequisites/feasibility | Prove first-start readiness and one local evidence-backed solve loop | G0 doctor + toolchain.lock; approved CTF runtime/profile; ctf-core + initial Misc/Web Skills; ACP Worker host; evidence tap; real screenshot collector; synthetic Misc and local Web E2E | Two tasks complete intake → CLI/headless action → captured result → real screenshot → candidate Flag → evidence-linked Writeup → export; integrity/recovery/fabrication tests pass | Not complete |
| P1 — Core solving loop | Reliable Manager + up to two Workers across five categories | Persisted FIFO; explore-before-escalate; five Skills; one CTF Tools MCP; dual network channels; scoped authorization; evidence Writeup | Three tasks prove two active + one FIFO; lifecycle/recovery pass; enabled categories solve synthetic/authorized tasks; every action evidenced; unauthorized access rejected | Planned |
| P2 — Isolation/permissions | Safely enable untrusted binaries and remote exploit modes | WSL2 Ubuntu; Docker per task; version-pinned non-root toolchain; filesystem/process/network/resource controls; default-deny egress; one-shot Pwn grants | Escape/host-file/unapproved-egress blocked; quotas enforced; grants expire and never cross tasks; enabled category gates pass | Blocked on environment gates |
| P3 — Authorized warm-up/freeze | Prove competition readiness | Signed/versioned doctor; competition CONDITIONS profile; startup/recovery/export manual; evidence quality review | Authorized warm-up tasks produce auditable Writeups; recovery/export pass; no fabricated steps or out-of-scope traffic | Planned |

**Naming warning:** engineering P0 Manager Core (P0-REVIEW-FIX-02) is not Product P0.

## 2. Engineering workstream A

| Milestone | Scope | Exit criteria | State |
|---|---|---|---|
| A0 — Repository/product baseline | North-star docs, decision log, security and handoff | One authoritative blueprint; status and next task explicit | Docs baseline on Draft PR #1 |
| A1 — P0 Manager Core source | Intake, immutable inputs, workspace, two-slot FIFO, state/recovery, Runner/ACP contracts, Mock Runner, SQLite, CAS, incremental Writeup | Exact source imported; target checks rerun; independent review passes | Accepted source candidate; migration pending |
| A2 — Real ACP Worker integration | Connect Runner to independent ACP Worker sessions with task work-root | Isolated create/prompt/permission/cancel/close/recovery smoke; no target access | Not started; ACP selected |
| A3 — Evidence automation | Real Host tool hooks, Collector, screenshots, artifact closure and provenance | Synthetic E2E; tamper/truncation/missing-evidence tests | Planned |
| A4 — Skills and tool adapters | ctf-core + Web/Misc/Crypto/Reverse/Pwn; native light tools; one CTF Tools MCP | Routing/playbook tests; MCP security contract S1–S7 passes | Planned |
| A5 — Isolation and authorization | WSL2/Docker executor, task mounts, quotas, target allowlists, one-shot grants | Escape/network/resource/shutdown/cleanup tests | Planned; depends on B |
| A6 — Writeup/export | Evidence-linked report, human draft/versioning, Flag states, ZIP/checksums | Consistent snapshot, provenance and default-redaction tests | Core primitives exist; full product flow unverified |
| A7 — Authorized practice pilot | Approved warm-up tasks per enabled category | Human sign-off, evidence audit and repeatability | Not started |

## 3. Environment workstream B

| Phase | Scope | Last known state |
|---|---|---|
| B0 | Windows baseline audit | Reported complete |
| B1 | WSL2 networking foundation | Reported PASS |
| B2 | Docker daemon proxy inheritance | Outstanding in last handoff; verify before action |
| B3 | Docker Engine install/start/hello-world | Not confirmed complete |
| B4 | Version-pinned CTF toolchain image | Not started |
| B5 | Per-task isolation acceptance | Not started |

The environment agent owns B tasks. Application development does not authorize OS mutations. Administrator actions, reboot, firewall/proxy/security settings, Docker daemon changes and policy exceptions require explicit user authorization and any required Safety Net approval. Do not infer policy approval from project-level authorization.

## 4. Dependency order — one active task at a time

1. **A0-2:** import the exact original 110-file P0 source archive into docs/project-baseline. Preserve north-star docs, apply only specified path redactions/renames, and rerun target-repository checks.
2. Independently review the migration PR; keep draft/open/unmerged until accepted.
3. **A2:** implement real ACP Worker adapter against frozen Runner contract. No target traffic required for smoke tests.
4. **A3:** connect actual Host tool events/results and screenshots to the authoritative Collector.
5. **A4:** implement Skills and one structured CTF Tools MCP server.
6. **A5/B:** complete environment readiness and OS/container isolation before untrusted binaries or remote exploit execution.
7. **A6/A7:** complete Writeup/export E2E and authorized warm-up pilot.

Do not bundle A2–A7 into source migration. If an acceptance test exposes a migration defect, stop and classify it before expanding scope.

If WSL2/Docker readiness is delayed, enable only modes whose current toolchain/isolation gates pass. Do not simulate capability.

## 5. Definition of done

Every task records task ID/phase, product outcome, scope/non-goals, starting branch/HEAD, changed files, preserved interfaces, actual commands/outputs/exit codes, skips/failures, security evidence, artifact hashes, limitations, updated STATUS.md and reviewer disposition (PASS / PASS-WITH-NOTES / REQUEST-CHANGES / BLOCKED).
