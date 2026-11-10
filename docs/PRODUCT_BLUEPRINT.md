# Product Blueprint — CTF Solver

> **This is the product north star.** When implementation details, agent suggestions or later refactors conflict with this document, stop and raise an explicit decision request. Do not silently move the product toward a generic chat wrapper, a second agent platform or a second evidence store.

## 1. Product definition and intended experience

CTF Solver is a **semi-autonomous, local-first CTF workbench** with one unified Harness entry point. The user submits challenges, provides or confirms scope, approves sensitive actions, confirms Flags and reviews the resulting evidence-backed Writeup. The system manages routine work: task intake, category routing, bounded Worker execution, progress collection, evidence capture, recovery and export.

It is not:
- a generic chatbot that merely gives solving suggestions;
- a fully autonomous bot that attacks arbitrary targets;
- a second standalone web platform duplicating the Harness UI;
- a Writeup generator that invents a plausible solution from a final Flag;
- a collection of disconnected Skills/MCP servers with competing task state.

**The product outcome is a repeatable challenge-to-Writeup workflow whose actions, outputs and claims can be audited.**

## 2. Primary architecture decision

The feasibility audit's recommended target is **Scheme A: Skill as the brain, MCP as the hand, Cordis as the thin skeleton**, using Harness native capabilities wherever they already exist and reusing the existing CTF Workbench evidence backend.

### 2.1 Target topology

    DeepSeek Harness — one window / Web GUI
      Manager Session (ctf-manager preset + ctf-core Skill)
        |-- asks user for missing challenge/scope information
        |-- reads task state and queue from the Manager Core service
        |-- relays approvals; never self-approves
        |-- reports progress, blockers, outcomes and Writeup readiness
        |
        +-- Worker Session A (native subagent, task-specific cwd/preset)
        +-- Worker Session B (native subagent, task-specific cwd/preset)
                 |
                 +-- ctf-core + one category Skill
                 +-- Harness native tools for ordinary file/shell/web work
                 +-- ctf-tools MCP for justified heavy/fixed-version tools
                 +-- every relevant tool action -> Cordis evidence tap
                                      |
                 Thin Cordis host adapter / hooks
                    |-- connects Harness lifecycle to Manager Core
                    |-- forwards worker events and permission requests
                    |-- captures evidence and screenshot metadata
                    |-- does not create a second queue or event database
                                      |
                 Existing CTF Workbench evidence backend
                    |-- SQLite append-only event/hash-chain store
                    |-- content-addressed Artifact Store (CAS)
                    |-- safe ZIP import and immutable originals
                    |-- evidence-linked Writeup and ZIP export
                                      |
                 WSL2 / Docker execution boundary (later security gate)
                    |-- per-task execution environment
                    |-- filesystem/process/network/resource controls

The UI remains the Harness GUI. A small read-only queue/evidence panel may be considered later, but it must not become a parallel application or a second source of truth.

### 2.2 Worker host choice and profile prerequisite

The audit recommended a dedicated **ctf profile (W1)**:
- remove the experimental Team bundle that masks native subagent-control tools;
- retain native subagent creation and control, Skills, jobs, filesystem and approval tools;
- configure maxActiveSubagents=2 (or a deliberately reviewed value) and maxDepth=1;
- enable only required optional capabilities, such as allowed presets, session query and MCP client;
- use one native subagent session and one task-specific cwd per challenge.

The Team-based W2 option is rejected for the first release because teammates share a cwd and do not provide the required per-worker cwd/model/preset control. A task-board-per-session W3 is only a fallback for later evaluation, not the default.

**Profile changes are user-approved environment/configuration changes.** Do not edit the installed Harness profile or enable capabilities without explicit approval.

## 3. Single responsibility for each layer

| Layer | Owns | Must not own |
|---|---|---|
| Harness GUI / Manager Session | Unified interaction, user questions, approvals relay, summaries and handoff | Heavy solving, a duplicate task database |
| Manager Core service | Task state machine, authoritative queue/FIFO admission, Worker lifecycle contract, recovery and task-to-evidence links | Category-specific solving logic |
| Harness native subagents | Worker sessions, isolated context, task cwd and native lightweight tools | Cross-task state or self-granted permissions |
| Core Skill | Intake checklist, task routing, workflow, evidence discipline, Manager/Worker protocol and reporting format | Executable permission enforcement or storage |
| Five category Skills | Domain knowledge and playbooks for Web, Misc, Crypto, Reverse and Pwn | Queue, secrets store, tool implementation or authorization |
| Native Harness tools | Ordinary filesystem, shell, jobs and web operations | Heavy fixed-version toolchain wrappers already assigned to MCP |
| One CTF Tools MCP server | Structured access to heavy/fixed-version tools such as Ghidra Headless, GDB batch/MI, pwntools, SageMath, controlled container execution and screenshot capture | Independent authorization policy or evidence database |
| Thin Cordis plugin | Host hooks, Worker lifecycle bridge, permission relay, evidence tap and optional screenshot action | A second Agent loop, second scheduler, second task store or duplicate evidence store |
| Evidence backend | SQLite event/hash-chain authority, CAS bytes, safe imports, Writeup rendering and export | Agent orchestration or free-form solving |
| WSL2/Docker executor | OS-level filesystem/process/network/resource isolation | Product decisions or task state authority |

**Classification rule:** if it teaches the model how to solve, it belongs in a Skill; if it executes a structured heavy tool, it belongs in the CTF Tools MCP; if it must happen automatically or be enforced across sessions, it belongs in Manager Core / the thin Cordis bridge / OS execution boundary; if it records evidence, it belongs in the existing evidence backend.

## 4. Challenge lifecycle — required end-to-end behavior

1. **Submit:** user provides challenge name, category, description, optional screenshot/ZIP, container flag, target/range address and ports.
2. **Intake:** Core Skill checks missing information and asks the user. The import service validates the ZIP, preserves original bytes immutably, hashes them, creates a manifest and provisions the task workspace.
3. **Declare scope:** target and authorization are recorded separately. A target address in a challenge statement is not proof of permission.
4. **Queue:** Manager Core persists task state and admits at most two active tasks. Others wait in FIFO order.
5. **Spawn:** Manager creates one Worker session for the task, with the category preset and task-specific cwd. Each Worker sees only its own task context and workspace contract.
6. **Route:** Worker loads ctf-core plus exactly the relevant category Skill. The Manager does not load all five specialist playbooks into its own context.
7. **Solve loop:** Worker forms a hypothesis, uses native tools or the CTF Tools MCP, observes the real result, updates its hypothesis and reports meaningful progress/blockers to Manager.
8. **Authorize:** remote access, Pwn exploitation and other sensitive actions require a narrow, explicit, task-bound, time-bounded approval. Worker requests; human approves; the policy layer enforces. No approval is inherited across tasks.
9. **Capture evidence:** tool action/result, timestamps, task/worker identity, exit status where available, artifact digest and screenshot provenance are captured as the work happens. Worker self-report alone is not proof that a command ran.
10. **Complete/recover:** Manager records solved, unsolved, failed or cancelled. Interrupted work retains its evidence and can be resumed or rebuilt according to the lifecycle contract.
11. **Confirm Flag:** distinguish candidate, user-confirmed and externally/platform-verified. A user confirmation is not a platform verification.
12. **Writeup/export:** generate Markdown from persisted events and artifacts; each claim links to evidence. Missing steps remain explicitly missing. User reviews/edits the draft, then exports a ZIP with manifest and SHA256SUMS.

## 5. Product invariants

- **One challenge = one Worker = one task workspace.** No shared cwd across active challenges.
- **At most two active Workers; FIFO beyond capacity.** Pause is not a free slot in the current P0 contract; paused work retains its slot. Slot release follows confirmed shutdown.
- **Manager does not do heavy solving.** It coordinates, asks, approves via the human, monitors and reports.
- **One approval path.** No Worker self-approval; permissions are explicit and scoped to a task/target/action/expiry.
- **Evidence is append-only.** Corrections append new records and refer to the original; they do not rewrite history.
- **SQLite is the only authoritative task/event state store.** Do not add queue.json, a second SQLite database or JSONL as another source of truth.
- **CAS is the authoritative byte store.** Originals remain immutable; agent payloads and candidate Flag plaintext do not enter SQLite event payloads.
- **Writeup is evidence-derived.** Never invent commands, outputs, screenshots, Flag verification or missing steps.
- **Directory isolation is not a sandbox.** Real untrusted binaries require OS/container enforcement.
- **Local-first.** Real challenge material, credentials, Flags, raw logs and Writeups do not enter this public repository.
- **Scope-limited use only.** No arbitrary Internet targets, competition infrastructure, scoreboard, registration or submission endpoint attacks.

## 6. Product phases from the feasibility audit

These are **product release phases P0–P3**. They are not the same numbering as the later engineering milestone called “P0 Manager Core”.

### Product P0 — prerequisites and feasibility proof

Deliver:
- dedicated ctf profile (after user approval);
- thin ctf-core Skill plus thin ctf-misc and ctf-web;
- evidence service reusing the existing Workbench store/artifact/import/export code;
- Cordis evidence tap for native/MCP tool actions;
- screenshot collector recording window/process/time/hash metadata;
- first-run doctor snapshot for tool versions, WSL/container availability, disk and relevant host constraints.

Hard acceptance: two synthetic challenges (Misc file analysis and local Web lab) complete intake → analysis → tool action → real screenshot → candidate Flag → evidence-linked Writeup → export. Verify hash chain, tamper/truncation detection, missing-evidence marking, screenshot provenance, restart continuity and rejection of fabricated Writeup steps.

### Product P1 — core solving loop

Deliver:
- Manager/Core workflow and thin Cordis host bridge;
- persisted FIFO scheduler with no more than two active Workers;
- Manager reporting, approval relay, stalled-task reporting and recovery protocol;
- five Skills (Web/Misc/Reverse first; Crypto/Pwn playbooks added with clear environment gates);
- one CTF Tools MCP v1 for justified heavy tools;
- evidence-linked Writeup generator and scoped authorization ledger.

Acceptance: submit three tasks, observe two active and one FIFO queued; exercise pause/resume/query; solve at least three authorized/synthetic tasks per enabled category; all tool calls have evidence; unauthorized remote access is rejected and recorded; Worker failure preserves evidence.

### Product P2 — isolation and permissions

Depends on explicit approval for WSL2/administrator/reboot work.

Deliver:
- WSL2 Ubuntu 24.04 and versioned CTF toolchain;
- Docker Engine inside WSL2; one container per task;
- execution adapter with Windows/WSL/container modes and directory/file/process/network controls;
- default-deny network with exact domain or IP+port allowlists;
- one-shot Pwn grant bound to target, time window and interaction method;
- screenshot attribution for multiple Workers and headless Web evidence.

Acceptance: block container escape, host-file access and unapproved egress; enforce CPU/memory/disk limits; prove one-shot grants expire and never cross tasks; exercise all five categories including synthetic Pwn/Crypto.

### Product P3 — competition validation and freeze

Deliver:
- signed/versioned environment doctor snapshot and drift alerts;
- competition CONDITIONS profile (rules, authorization, Flag format, AI disclosure);
- operations guide for startup, recovery, evidence export and Flag-state confirmation;
- post-event Writeup evidence-quality review.

Acceptance: at least one authorized warm-up challenge produces an auditable Writeup; interrupted task recovery succeeds; export hashes verify; no fabricated steps or out-of-scope traffic; user actions are limited to submit, approve, confirm Flag and review reports.

If WSL2 approval or Docker readiness is delayed, Misc/Web/Reverse static analysis may proceed within the safe Windows capability envelope. Pwn/Crypto requiring Linux toolchains remain explicitly blocked; do not simulate capability.

## 7. Architecture options considered

- **A — Skill + MCP + thin Cordis (selected target):** lowest duplication, Harness-native main path, one window, existing evidence backend reused.
- **B — Cordis platformization:** stronger control but 2–3× implementation/maintenance cost and deeper dependency on unstable host internals. Keep only the useful isolation/policy concepts for P2.
- **C — Harness + Agoni dual-agent system:** rejected for first release because it duplicates Agent loops, context, tools and evidence, while several domain implementations were incomplete in the audited version.
- **D — MCP maximum:** rejected as the primary architecture because MCP cannot automatically observe every Harness tool event; evidence would depend too much on Worker self-report. MCP remains appropriate for heavy tools and evidence queries.

## 8. How the later P0 Manager Core fits — no architecture drift

The accepted **P0-REVIEW-FIX-02 Manager Core** is an engineering foundation, not the whole product and not a replacement for the target topology above.

It provides a portable, deterministic core: TaskManager, two-slot FIFO Scheduler, per-task workspace, intake, Runner/ACP contracts, Mock Runner, SQLite event persistence, CAS evidence and incremental Writeup. It deliberately does **not** provide a real ACP process, live Harness Worker, ctf profile, five category Skills, real binary execution, target access, Docker isolation or a separate product UI.

Integration guardrails:
- Manager Core remains the authoritative task/queue/state owner. Cordis must bridge Harness events and Worker lifecycle into it; Cordis must not maintain a competing queue or database.
- Runner remains the runtime seam. The actual Host Worker adapter must be selected by a small A2 integration spike: compare the audited native subagent-control route with the P0 ACP adapter contract. Do not claim either is connected until a real, isolated smoke test proves create/update/permission/stop/close/recovery.
- The P0 ACP contract is not proof that ACP can provide every desired Host feature. Unsupported calls must remain explicit failures.
- Reuse the existing SQLite/CAS backend; no parallel evidence store.
- Keep the Harness GUI as the product entry point; do not expand the existing frontend into a second application.
- Preserve the original product P0–P3 acceptance gates. Passing Manager Core tests alone does not complete product P0.

## 9. Change control

Any proposed change to the unified Harness entry point, Scheme A layer responsibilities, five categories, one-task/one-Worker model, two-worker FIFO limit, single SQLite/CAS evidence authority, evidence-derived Writeup, human approval boundary, or product P0–P3 gates requires:
1. a written rationale and alternatives;
2. a decision-log entry;
3. impact on security, evidence and acceptance criteria;
4. explicit user approval before implementation.

Until then, this blueprint is the product north star.
