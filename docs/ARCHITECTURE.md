# Architecture and Responsibility Boundaries

This document separates the **current P0 Manager Core implementation** from the **target CTF Solver product architecture**. PRODUCT_BLUEPRINT.md is the product authority.

## 1. Current P0 Manager Core

```text
Existing Workbench API / UI
  -> TaskManager
      -> IntakeService
      -> WorkspaceService
      -> two-slot FIFO Scheduler
      -> Runner protocol
          -> ACPClient adapter contract
          -> MockRunner / MockACPClient (P0 only)
      -> SolverStore
          -> SQLite task/event state and event links
          -> existing ArtifactStore / SHA-256 CAS
      -> WriteupService
```

Source package: `backend/app/solver/`; public entry point: `app.solver.manager.TaskManager`.

P0 supplies task intake, immutable input handling, per-task workspace construction, persisted two-slot FIFO scheduling, Worker lifecycle contracts, deterministic Mock Runner behavior, task recovery metadata, SQLite event persistence, CAS artifact storage and incremental Writeup generation.

**P0 does not prove** a real ACP process, live Harness Worker, live model provider, real binary execution, target connectivity, Docker isolation or competition-platform Flag verification.

## 2. Target product topology

Selected target: Scheme A — Skill as the brain, MCP as the hand, Cordis as the thin skeleton.

```text
Harness GUI / Manager Session
  -> Manager Core / TaskManager (only task and queue authority)
      -> Runner / ACP adapter
          -> independent ACP Worker Session per task
             workspace root = tasks/<task_id>/work
      -> human approval relay
      -> progress and outcome reporting

Worker bootstrap
  -> ctf-core + one category Skill
  -> native CLI/headless tools for ordinary work
  -> one CTF Tools MCP for justified heavy tools

Thin Cordis / Host bridge
  -> lifecycle and permission-request hooks
  -> evidence tap / screenshot metadata
  -> no queue, task DB or second Agent loop

Evidence backend
  -> SQLite: append-only events, task/worker links, state
  -> CAS: immutable bytes and sensitive payload objects
  -> evidence-derived Writeup / ZIP export

Product P2 execution boundary
  -> WSL2 / Docker with per-task filesystem/process/network/resource controls
```

Harness GUI remains the single user-facing entry point. Do not create a parallel full-stack UI.

## 3. Worker host and workspace boundary

The accepted Worker carrier is **one independent ACP session per task**, created with that task's `work/` directory as the session workspace root. The session workspace is established at creation. A child Agent's `cwd` does not change the parent session writable root and is not an acceptable substitute.

Workspace layout:
- `input/`: immutable original attachments / verified read-only copies;
- `work/`: Worker writable root;
- `output/`: Manager-collected outputs;
- `scripts/`: controlled task scripts where the Manager contract permits;
- `writeup/`: derived report material, not a Worker-controlled evidence authority.

Worker path resolution must not expose input/output/writeup as writable destinations. `collect_output` accepts only paths under task work/, then copies bytes into CAS/output with Worker attribution. Re-hash immutable inputs before/after use.

ACP v1 methods modeled by the contract:
- `session/new`, `session/list`, `session/resume`, `session/prompt`,
- `session/cancel`, `session/close`, `session/request_permission`.

The ACP Worker Session does not expose terminal, ask_user, todo/plans, command APIs, elicitation, arbitrary extra writable directories or per-session presets; it also does not provide process suspend/resume or OS process kill. Manager owns user interaction and task tracking; the Runner/approved tool wrapper provides execution surfaces; Worker bootstrap must supply ctf-core plus exactly one category Skill through a verified session-bootstrap mechanism. Never assume a per-session preset filter exists. Missing interaction features belong in Manager/Runner/wrapper only after verified implementation. Session resume is not proof that a dead OS process was restored.

## 4. Component ownership

| Concern | Owner | Must not be duplicated in |
|---|---|---|
| Task lifecycle/orchestration | TaskManager | UI, Worker prompt, ad-hoc SQL |
| Admission and FIFO | scheduler.py + Manager | queue JSON, Cordis scheduler, frontend |
| Persistence/state transitions | SolverStore / SQLite | second JSONL or parallel DB writer |
| Worker lifecycle abstraction | Runner protocol | Manager-specific runtime calls |
| ACP wire surface | ACPClient adapter | category Skills |
| Per-task workspace | workspace.py | arbitrary Worker path construction |
| Attachment intake/integrity | intake.py + ArtifactStore | mutable original copies |
| Evidence bytes | CAS | duplicate blob stores |
| Writeup rendering | WriteupService from persisted facts | free-form model reconstruction |
| Category knowledge | Core + five category Skills | MCP server / persistence |
| Heavy fixed-version tools | one CTF Tools MCP server | prompt text or duplicate wrappers |
| Host hooks and evidence tap | thin Cordis/Host bridge | second queue or event database |
| Hard execution isolation | WSL2/Docker executor | Skill text, cwd or ACP session alone |

## 5. Manager, queue and lifecycle invariants

- One task maps to one challenge and at most one active Worker generation.
- Default concurrency is two; hard maximum is two; further tasks queue FIFO.
- Manager is the sole task/queue/state authority. Cordis bridges host events/permissions but does not own a second scheduler.
- Manager handles intake, status, progress, approval relay, recovery and reporting. Worker performs heavy challenge analysis.
- Paused is non-terminal and retains its slot.
- Cancel is terminal. Queued cancellation creates no Worker session.
- Worker stop must be confirmed before close, terminal state persistence and slot release.
- If stop is unconfirmed, preserve Worker mapping and slot. If close fails, mark cleanup pending, preserve slot and allow explicit cleanup retry.
- On restart, reconstruct task/evidence state. Do not claim Agent process restoration. Resume only if ACP confirms the session is resumable; otherwise create a new Worker generation and preserve prior evidence.

## 6. Evidence and Flag integrity

- SQLite is the only authoritative task/event/state store.
- CAS is the authoritative byte store; JSONL/Markdown/ZIP are derived exports only.
- Every event uses authoritative task/worker IDs and participates in the integrity chain.
- Agent event payload bytes are stored in CAS; SQLite contains only digest/size/reference metadata.
- Worker event whitelist excludes system-owned approval, task-state, worker-close and verified-Flag transitions.
- Candidate Flag values travel through the structured Manager API, not arbitrary Agent event payloads. Plaintext is a protected CAS artifact; SQLite stores only digest, length and reference.
- Flag states are candidate → user-confirmed → externally verified. User confirmation is not platform verification; only a real FlagVerifier may produce verified state.
- Writeup facts cite event/artifact IDs. Missing evidence is a visible gap. Export excludes/redacts sensitive Flag and Agent payload artifacts by default.

## 7. Execution and tool boundaries

### 7.1 CLI/headless first
Workers use CLI/headless tools by default. GUI applications may be launched for analysis and screenshot evidence; autonomous mouse/keyboard operation is not required for the first release.

### 7.2 Skills / native tools / MCP / Cordis
- Core Skill: common workflow, evidence, authorization request and escalation rules.
- Five category Skills: specialist knowledge only.
- Native Harness tools: ordinary file/text/CLI/headless operations and auxiliary research.
- One CTF Tools MCP: structured heavy-tool operations (Ghidra Headless, GDB, pwntools, SageMath, controlled container execution and screenshot capture where justified).
- Thin Cordis/Host bridge: lifecycle, permission relay, evidence hooks and screenshot metadata. No duplicate Agent loop, scheduler or evidence store.

MCP runs outside the Harness shell sandbox/approval boundary. Every MCP operation independently validates task ID, Worker ID, authorization, network scope, schema, timeout, output size and isolation. No arbitrary shell endpoint.

### 7.3 Separate network channels
- Auxiliary research: ordinary web tools for documentation/public research; challenge targets are blocked.
- Challenge range: structured `ctf-range-*` APIs with per-call task/target/port/protocol/action/window checks.
- Never route target traffic through generic web tools or unscoped shell.

## 8. G0 environment readiness

Before formal solving, run the first-start environment doctor, verify the required toolchain for all five categories, install missing tools only through the approved environment workflow, run self-tests and freeze versions in `toolchain.lock`. Missing tools during a solve trigger installed-alternative checks and an environment blocker report—not ad-hoc host installation by a Worker.

Warm-up challenges may add tools to the inventory. Additions require review, installation through the environment workflow and a new doctor/lock record.

## 9. Isolation warning

A task workspace, separate ACP session, Skill policy or MCP schema is **not by itself an OS sandbox**. Real untrusted binary/exploit execution requires the Product P2 WSL2/Docker boundary: non-root execution, task-specific mounts, read-only originals, minimal capabilities, no host Docker socket mount, default-deny egress, resource/output/time limits, process-tree shutdown and verified cleanup.

Until those gates pass, only explicitly approved safe analysis modes are enabled. See SECURITY.md.
