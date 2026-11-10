# Product Blueprint — CTF Solver

> **Authoritative product north star.** Every implementation task follows this document. If a proposal conflicts with a settled decision, stop, record an ADR proposal and obtain explicit user approval before changing it.

## 1. Product definition

CTF Solver is a **semi-autonomous, local-first, evidence-first CTF workbench** with one unified Manager entry point. The user submits challenges, confirms authorization scope, approves sensitive actions, confirms candidate Flags and reviews evidence-backed Writeups. The system handles intake, category routing, bounded parallel work, progress collection, evidence capture, recovery and export.

It is not a generic chatbot, a prompt collection, an arbitrary Internet attack bot, a second Agent platform, or a Writeup generator that invents a solve path from a final Flag.

**Product outcome:** a repeatable challenge-to-Writeup workflow in which every claimed action and result is traceable to persisted evidence.

## 2. Settled product decisions

1. **One task, one Worker, one workspace.** A challenge never shares its active Worker context or workspace with another challenge.
2. **Two active tasks maximum.** Default concurrency is two; the hard limit is two. The third and later tasks wait FIFO. This is a capacity guard, not the product's main selling point.
3. **Manager coordinates; Worker solves.** Manager handles intake, queue, lifecycle, approvals, recovery and reporting. It does not absorb specialist analysis.
4. **CLI/headless first.** Workers must solve through command-line and headless tools. GUI applications may be opened and real screenshots collected, but autonomous clicking through every GUI control is not a first-release requirement.
5. **Autonomous exploration before escalation.** A Worker checks available tools, tries suitable installed alternatives and explores reasonable local approaches before reporting that it is stuck. It asks the user only when missing information, authorization, a decision or a genuine blocker prevents safe progress.
6. **Environment prepared before solving.** First startup runs the G0 environment-readiness gate. Required tools are installed/configured in the approved environment, verified and frozen in `toolchain.lock`. Workers do not install software or alter the Windows host during a solve.
7. **Skills are knowledge, not security.** Skills define solving methods and behavior; tool policy, approval checks and OS/container isolation enforce permissions.
8. **SQLite + CAS are the sole evidence authority.** SQLite is the authoritative task/event/state store; CAS is the authoritative byte store. JSONL/Markdown/ZIP are derived outputs only. No dual-write or parallel queue/database.
9. **Writeups are evidence-derived.** Missing steps remain visibly missing. A Flag or model statement alone is not proof.
10. **Human approval for sensitive actions.** A Worker may request but never grant its own permission.
11. **Public Git contains code, generic docs and synthetic fixtures only.** Real challenge files, Flags, credentials, raw logs, private target data, screenshots and real Writeups remain local.

## 3. Target architecture — Scheme A

The selected architecture is **Skill as the brain, MCP as the hand, Cordis as the thin skeleton**, using Harness-native capabilities where appropriate and reusing the existing CTF Workbench evidence backend.

```text
Harness — one user-facing window
  Manager Session
    ├─ ctf-core workflow and policy guidance
    ├─ TaskManager: intake, persisted queue, lifecycle, recovery, reporting
    ├─ human approval relay; never self-approves
    └─ Runner / ACP adapter
         ├─ independent ACP Worker Session for task A
         │    └─ workspace root = tasks/<task_id>/work
         └─ independent ACP Worker Session for task B
              └─ workspace root = tasks/<task_id>/work

Worker bootstrap
  ├─ ctf-core + exactly one category playbook
  ├─ Harness/native CLI and headless tools for ordinary work
  └─ one CTF Tools MCP server for justified heavy/fixed-version tools

Thin Cordis / Host bridge
  ├─ lifecycle and permission-request bridge
  ├─ evidence hooks and screenshot metadata
  └─ no second Agent loop, scheduler, task database or evidence store

Evidence backend
  ├─ SQLite: task state + append-only event/hash chain + task/worker links
  ├─ CAS: immutable input/output/agent-payload bytes by SHA-256
  └─ evidence-linked Writeup and ZIP export

Product P2 execution boundary
  └─ WSL2 / Docker: filesystem, process, network and resource isolation
```

The Harness GUI remains the user-facing entry point. Do not build a second full-stack UI.

### 3.1 Worker host decision: independent ACP Worker Session

The accepted Worker carrier is **one independent ACP session per task**, created with that task's work directory as the session workspace, conceptually:

```text
session/new { workspace: <task workspace>/work }
```

The runtime audit established that the session workspace at session creation is the relevant write-permission root. A child Agent's `cwd` changes its working directory but does **not** change the session writable root; therefore "one native subagent + one cwd" is not sufficient for the required per-task write boundary.

Consequences:
- Manager creates/tracks one ACP Worker session/generation per task through Runner.
- Worker workspace is exactly `tasks/<task_id>/work`; input originals, output collection and Writeup paths are not Worker write targets.
- ACP v1 is limited to verified methods: `session/new`, `session/list`, `session/resume`, `session/prompt`, `session/cancel`, `session/close`, `session/request_permission`.
- Do not invent ACP support for terminal APIs, plans, commands, elicitation, arbitrary extra writable directories, per-session presets, process suspend/resume or OS process kill. Manager/Runner/wrapper provides required orchestration outside ACP.
- ACP session recovery is not the same as restoring a dead Agent process. After restart, Manager reconstructs task state/evidence; resume only if ACP confirms the session is resumable, otherwise create a new Worker generation. Never claim a process was restored if it was not.
- A session workspace is not a kernel-enforced sandbox. Untrusted binaries/exploit code still require Product P2 isolation.

The earlier native-subagent-as-Worker proposal is superseded. Native Harness tools may still serve Manager and verified wrappers, but do not revert the Worker host to a shared-session subagent design.

### 3.2 Responsibility boundaries

| Layer | Owns | Must not own |
|---|---|---|
| Harness GUI / Manager Session | Unified interaction, user questions, approval relay, summaries and handoff | Heavy solving or duplicate task database |
| Manager Core / TaskManager | Authoritative task state, two-slot FIFO, Worker generations/lifecycle, recovery, task-to-evidence links | Category-specific solving |
| Runner / ACP adapter | Worker session create/prompt/permission/cancel/close and lifecycle result | Task policy or second scheduler |
| Core Skill | Intake workflow, routing, evidence discipline, escalation and Manager/Worker protocol | Permission enforcement, execution or persistence |
| Five category Skills | Web, Misc, Crypto, Reverse and Pwn playbooks | Queue, storage, secrets or authorization |
| Native Harness tools | Ordinary file/text/CLI/headless work and auxiliary research | Heavy fixed-version wrappers assigned to MCP |
| One CTF Tools MCP server | Structured heavy tools: Ghidra Headless, GDB, pwntools, SageMath, controlled container execution and screenshot capture where justified | Arbitrary shell, self-authorization or evidence DB |
| Thin Cordis / Host bridge | Host lifecycle hooks, permission relay, evidence tap, screenshot metadata | Second Agent loop, scheduler, task store or evidence store |
| Evidence backend | SQLite authority, CAS bytes, imports, event links, Writeup and export | Agent orchestration |
| WSL2/Docker executor | OS-level filesystem/process/network/resource isolation | Product state or task policy |

**Layer rule:** knowledge/playbooks → Skill; structured heavy execution → MCP; automatic host integration → thin Cordis/Host bridge; task/queue/lifecycle authority → Manager Core; evidence persistence → existing SQLite/CAS backend; hard isolation → OS/container executor.

### 3.3 Separate network channels

- **Auxiliary research:** ordinary Harness web search/fetch for public documentation and references. Challenge targets are blocked from this channel.
- **Challenge range:** target interaction uses structured `ctf-range-*` tools only, checking task ID, exact target, port/protocol, action and current authorization window on every call.
- The two channels are separated at the tool/policy boundary. No Worker may route target traffic through generic web tools or arbitrary shell.
- MCP processes run outside the Harness shell sandbox/approval mechanism. Each MCP operation must enforce its own task identity, permission, network scope, timeout, output and isolation checks.

## 4. First-start environment readiness — G0

G0 is a prerequisite **before the first formal solve**, not a tool-installation discussion repeated for every challenge.

1. Run a read-only doctor across the approved host/WSL/container environment.
2. Check required baseline tools for all five categories: executable presence, version, path, health/self-test and execution environment.
3. Install/configure missing baseline tools only through the separately approved environment workflow. Workers never install tools or modify Windows host state.
4. Run self-tests and freeze verified versions in `toolchain.lock` with a doctor report.
5. Enable only categories/modes whose toolchain and isolation prerequisites pass.
6. Warm-up challenges may reveal missing tools. Add them to the reviewed inventory, install through the environment workflow, rerun doctor and update the lock.

If a tool is unexpectedly unavailable during a solve, the Worker checks the frozen inventory and safe installed alternatives. If none can progress, it reports an environment blocker; it does not install software itself.

## 5. End-to-end task lifecycle

1. **Submit:** user provides challenge name/category/description, optional screenshot or ZIP, and declared target/container details.
2. **Intake:** validate metadata/archive; preserve original bytes immutably; hash and manifest; provision task workspace.
3. **Scope:** record authorization separately from target details. A target in a prompt or attachment is not proof of permission.
4. **Schedule:** Manager admits at most two active tasks; later tasks queue FIFO.
5. **Create Worker:** Runner creates one ACP session with that task's work directory as workspace root.
6. **Route:** bootstrap ctf-core and only the matching category playbook.
7. **Explore and solve:** Worker uses CLI/headless tools first, captures actual results and updates hypotheses.
8. **Escalate only when blocked:** after reasonable autonomous exploration and installed-tool alternatives, report the exact blocker, attempts and evidence to Manager.
9. **Authorize:** Manager relays request; human approves; policy enforces. No self-approval or permission inheritance across tasks.
10. **Capture:** record action/result, timestamps, exit status where available, task/worker identity, digest and screenshot provenance.
11. **Close/recover:** stop confirmation precedes close and slot release. Unconfirmed stop or failed close keeps the Worker mapped and slot occupied; cleanup-pending is visible and retryable.
12. **Confirm/deliver:** preserve Flag state, generate evidence-derived Writeup, review gaps, export manifest + SHA-256 checksums.

### 5.1 Concurrency and lifecycle

- Default active task count: **2**; hard maximum: **2**.
- Third and later tasks enter persisted FIFO.
- One task has at most one active Worker generation.
- Paused tasks retain their slot in the current contract.
- Cancel is terminal. Queued cancellation creates no Worker session.
- Running Worker releases its slot only after Runner confirms stop and the session is closed.
- If stop is unconfirmed or close fails, retain slot and Worker mapping; record a safe diagnostic and require cleanup retry.

## 6. Solving behavior by category

All categories use the common Core Skill workflow. Category Skills supply specialist knowledge, not permissions.

| Category | Agreed strategy |
|---|---|
| **Misc** | Inspect file type, metadata, containers, encodings, steganography and authorized traffic captures. For encrypted archives, analyze format/metadata/password hints first, then make bounded local password attempts. Ask the user only after reasonable local paths are exhausted. |
| **Crypto** | Combine mathematical reasoning with reproducible tool calculations. Seek a mathematical breakthrough first; use Python/Sage or other verified tools to calculate, test and validate. |
| **Reverse** | Static analysis first (strings, headers, disassembly, decompilation, control/data flow). Dynamic execution only inside an accepted independent isolation environment. |
| **Pwn** | Local analysis/testing first. Remote exploitation requires a one-time human grant scoped to the same task, target and time window. After approval, autonomously validate within that scope. Completion/expiry invalidates the grant; it never transfers. |
| **Web** | Source, configuration and local reproduction first. Remote requests only to the explicitly authorized challenge target and approved action; no broad scanning/enumeration. |

## 7. Authorization and safety

- Worker requests permission; it cannot grant permission to itself.
- Approval is separate from execution and checked at the enforcement point on every protected operation.
- Pwn uses **window semantics**: one approval permits a continuous bounded sequence for the same task and target during the approved time window. Completion/expiry invalidates it.
- Generic research and challenge-range access are separate channels.
- No arbitrary Internet attacks, competition-platform/scoreboard/registration/submission attacks, Flag-submission brute force, denial of service, broad scanning, cross-team data sharing or out-of-scope pivoting.
- Organizer rules and explicit scope take precedence.
- Directory/session boundaries are not OS sandboxing. Untrusted binaries/exploit code remain disabled until Product P2 isolation gates pass.

## 8. Evidence, Flag and Writeup contract

### 8.1 Single authority
- SQLite is the only authoritative task/event/state store.
- CAS is the authoritative byte store, keyed by SHA-256.
- JSONL, Markdown, screenshots, manifests and ZIPs are derived outputs only; no dual-write.
- Events use authoritative task/worker IDs and append-only integrity chaining.
- Original inputs are immutable; workspace copies are read-only and re-hashed.
- Agent payload bytes go to CAS; SQLite stores only non-sensitive digest/size/reference metadata. Do not copy raw Agent payload into event JSON.
- Worker event whitelist excludes system-owned approval, task-state, worker-close and verified-Flag transitions.

### 8.2 Flag states
1. **Candidate:** Manager stores the value as a protected CAS artifact and records only digest/length/reference in SQLite.
2. **User-confirmed:** user confirms the candidate; this is not external/platform verification.
3. **Verified:** only a real configured FlagVerifier may produce this state, with verifier source/reference.

Never store Flag plaintext in SQLite events or normal logs. Export excludes/redacts `flag_capture` and sensitive `agent_event_payload` artifacts by default.

### 8.3 Writeup
Writeup is generated only from persisted task events and referenced evidence. Every factual command, observation, transformation and conclusion cites event/artifact IDs. Distinguish fact, hypothesis, failed attempt, human note and retrospective note. Missing evidence is a visible gap; never invent a command, output, timestamp, tool, screenshot or intermediate step. Solve status and Writeup completeness are separate.

Export Markdown first, then a reviewed ZIP with manifest and SHA-256 checksums. Sensitive Flag and Agent payloads are excluded/redacted by default.

## 9. Product release gates

These are **Product P0–P3**, not the engineering milestone called P0 Manager Core.

### Product P0 — prerequisites and feasibility proof
Deliver:
- approved CTF runtime/profile setup;
- G0 doctor, baseline toolchain inventory and `toolchain.lock`;
- thin ctf-core and initial Misc/Web Skills;
- Manager Core connected to selected ACP Worker Session host;
- evidence tap for real Harness/ACP tool actions;
- real screenshot collector with window/process/time/hash attribution;
- one synthetic Misc task and one local Web task end to end.

Exit gate: both tasks complete intake → Worker analysis → captured CLI/headless action/result → real screenshot → candidate Flag → evidence-linked Writeup → export. Hash-chain/tamper/truncation, missing-evidence, screenshot provenance, restart continuity and fabricated-step rejection pass.

### Product P1 — core solving loop
Deliver reliable Manager/ACP Workers and persisted two-slot FIFO; autonomous explore-before-escalate; five Skills; one CTF Tools MCP v1; dual network channels; scoped authorization ledger; evidence-linked Writeup/export.
Exit gate: three tasks prove two active + one FIFO queued; stop/close/cleanup/recovery pass; enabled categories solve synthetic/authorized tasks; every tool action has evidence; unauthorized remote access is rejected and recorded.

### Product P2 — isolation and permissions
Depends on separate approved WSL2/Docker work.
Deliver WSL2 Ubuntu, Docker per task, version-pinned non-root toolchain, task mounts, minimal capabilities, no host Docker socket mount, default-deny egress, exact target allowlists, resource/output/time limits, one-shot Pwn grants and verified cleanup.
Exit gate: escape/host-file/unapproved-egress tests fail closed; quotas hold; grants expire and never cross tasks; synthetic Pwn/Crypto and all enabled categories pass.

### Product P3 — authorized warm-up validation and freeze
Deliver signed/versioned doctor snapshot and drift detection; competition CONDITIONS profile; startup/recovery/export manual; evidence/Writeup quality review.
Exit gate: authorized warm-up tasks per enabled category produce auditable Writeups; recovery/export pass; no fabricated steps or out-of-scope traffic.

If Docker readiness is delayed, only modes whose toolchain/isolation gates pass may proceed. Misc/Web/Reverse static analysis may be enabled in the safe Windows envelope; dynamic Reverse, Pwn and Linux-dependent Crypto remain blocked where required isolation/tooling is absent. Never simulate capability.

## 10. Engineering and environment tracks

Engineering: **A0** repository/product baseline and source migration; **A1** P0 Manager Core; **A2** real ACP Worker integration; **A3** evidence hooks/Collector/screenshots; **A4** Skills + CTF Tools MCP; **A5** authorization + OS isolation; **A6** Writeup/export E2E; **A7** authorized practice pilot.

Separate environment workstream **B0–B5** covers Windows audit, WSL2 networking, Docker proxy inheritance, Docker Engine validation, toolchain image and isolation acceptance. Environment work must not silently alter application contracts. OS/admin/firewall/proxy/security/reboot actions require explicit user authorization and any required policy approval.

## 11. Change control

Changes to the unified Harness entry point, Scheme A responsibilities, ACP Worker decision, five categories, one-task/one-Worker model, two-worker hard limit, G0, CLI-first strategy, single SQLite/CAS authority, approval boundary, category strategy or Product P0–P3 gates require written rationale, alternatives, decision-log entry, security/evidence/acceptance impact and explicit user approval.

For every new conversation or agent, read README and AGENT_WORKFLOW.md. **The repository is the source of truth; chat history is not a substitute.**
