# Roadmap and Phase Gates

This roadmap has **three separate tracks**. Do not confuse them:
- **Product phases P0–P3** define what the user-facing CTF Solver must deliver.
- **Engineering milestones A0–A7** describe implementation of the reusable code foundation and integrations.
- **Host-environment phases B0–B5** prepare WSL2/Docker and isolation. Environment setup is not product capability by itself.

The product phase definitions and hard acceptance criteria are authoritative in PRODUCT_BLUEPRINT.md.

## 1. Product roadmap P0–P3 (authoritative)

| Product phase | Goal | Main deliverables | Exit gate | Current state |
|---|---|---|---|---|
| P0 — Prerequisites and feasibility proof | Prove a real single-task loop on the approved Windows capability envelope | Dedicated ctf profile; ctf-core + thin Misc/Web Skills; reused evidence service; Cordis evidence tap; real screenshot collector; environment doctor | Two synthetic tasks complete intake → tool action → real screenshot → evidence-linked Writeup → export; chain/tamper/truncation/restart/fabrication tests pass | **Not complete** |
| P1 — Core solving loop | Manager + up to two Workers solve enabled categories | Persisted FIFO; Manager/Worker protocol; five Skills; one CTF Tools MCP; evidence-linked Writeup; scoped authorization ledger | Three tasks prove two active + one FIFO; enabled categories pass synthetic/authorized tasks; every tool action evidenced; unauthorized access rejected | Planned |
| P2 — Isolation and permissions | Safely enable untrusted binaries, Pwn and Crypto workflows | WSL2 Ubuntu; Docker per task; execution adapter; filesystem/process/network/resource controls; default-deny egress; one-shot Pwn grants | Escape/host-file/egress blocked; quotas enforced; grants expire and never cross tasks; all five categories exercised | Blocked on environment and approvals |
| P3 — Competition validation and freeze | Prove warm-up competition readiness | Signed environment doctor; competition CONDITIONS profile; operations/recovery manual; Writeup quality review | Authorized warm-up challenge produces auditable Writeup; recovery and ZIP checks pass; no fabricated evidence or out-of-scope traffic | Planned |

**Important naming distinction:** “P0-REVIEW-FIX-02 Manager Core” is an engineering code milestone. It does not mean Product P0 above is complete.

## 2. Engineering workstream A — code and integration

| Milestone | Scope | Exit criteria | Status |
|---|---|---|---|
| A0 — Repository/product baseline | Product blueprint, charter, architecture, decisions, security, roadmap and agent handoff rules | Repository docs agree; one north star and one next task | Documentation baseline on PR #1; source import pending |
| A1 — P0 Manager Core | Intake, immutable inputs, per-task workspace, two-slot FIFO, persisted state/recovery, Runner/ACP contracts, Mock Runner, SQLite events, CAS and incremental Writeup | Historical candidate review and demo accepted; re-run after import | **Accepted source candidate**: P0-REVIEW-FIX-02 |
| A2 — Real Host Worker adapter | Bind the Manager Runner contract to the selected Harness Worker host (native subagent-control or ACP after an explicit spike) | Isolated real create/update/permission/stop/close/recovery smoke; no target access required | Not started; runtime choice remains open |
| A3 — Evidence automation | Connect real Host tool events/results to Collector; provenance, screenshots, artifact closure and audit completeness | Reproducible synthetic E2E; tamper/truncation and missing-evidence tests | Planned |
| A4 — Skills and tool adapters | Core Skill + Web/Misc/Crypto/Reverse/Pwn playbooks; native light tools; one CTF Tools MCP for heavy tools | Category routing tests; no duplicated permission/storage logic; adapter security review | Planned |
| A5 — Execution isolation and authorization | WSL2/Docker boundary, per-task mounts, quotas, target allowlists, one-shot grants and fail-closed behavior | Escape/network tests, CPU/memory limits, shutdown and cleanup tests | Planned; depends on B workstream |
| A6 — Writeup/export integration | Evidence-linked Writeup, human draft/version conflict handling, Flag states, final ZIP and SHA256SUMS | Consistent snapshot; no fabricated steps; hashes verified | Core primitives inherited; full product flow not yet verified |
| A7 — Authorized practice pilot | One approved practice challenge per category, then staged expansion | Human sign-off, evidence audit, no out-of-scope traffic, repeatable result | Not started |

## 3. Host-environment workstream B

| Phase | Scope | Handoff status |
|---|---|---|
| B0 — Windows baseline audit | Host capability and constraints inventory | Reported complete |
| B1 — WSL2 networking foundation | WSL2 mirrored networking and proxy path | Reported PASS in Phase 2B-1F |
| B2 — Docker daemon proxy inheritance | Ensure dockerd/containerd inherit the required proxy | **Known outstanding issue** in Phase 2B-1F handoff |
| B3 — Docker Engine validation | Official packages, daemon startup, image pull and hello-world | Not confirmed complete |
| B4 — CTF toolchain image | Version-pinned tools, non-root runtime, minimal capabilities and resource limits | Not started |
| B5 — Isolation acceptance | Mount, process, network, privilege, resource and cleanup tests | Not started |

Environment changes affecting OS features, reboot, administrator privileges, security controls, firewall, proxy, Docker daemon or persistent host configuration require explicit user approval. A narrowly scoped CC Safety Net exception request for Docker's public APT key was pending in the last recorded report; verify its current status before any action.

## 4. Dependency rules and immediate order

1. Complete A0-2: import the exact accepted P0 source snapshot into this repository, preserving the product blueprint and redacting local machine paths from historical logs.
2. Run the source migration tests in the target repository and record environment-specific skips/failures honestly.
3. Complete independent review of the migration PR before merge.
4. Then prepare the A2 Worker-host adapter spike against the frozen Runner/ACP contract. Do not assume native subagent and ACP are interchangeable.
5. Product P0 work (profile, Skills, evidence tap, screenshot and doctor) must be planned and accepted before claiming Product P0 complete.
6. Resolve B2/B3 only with explicit approval; do not let environment work silently modify Manager Core.
7. Do not start real target access or untrusted binary execution until the relevant Product P2 security gates pass.

If WSL2 approval or Docker readiness is delayed, Misc/Web/Reverse static analysis may proceed within the safe Windows capability envelope. Pwn/Crypto requiring Linux toolchains remain explicitly blocked; do not simulate capability.

## 5. Definition of done for every task

Every task must have scope and non-goals, exact files/interfaces changed, actual commands and output, security/failure-path analysis, evidence/checksums where relevant, updated STATUS.md and DECISIONS.md, and reviewer disposition (PASS / PASS-WITH-NOTES / REQUEST-CHANGES / BLOCKED).
