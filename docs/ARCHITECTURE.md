# Architecture and Responsibility Boundaries

## 1. Current architecture (P0 implementation boundary)

    User
      -> Existing Workbench UI / API
      -> TaskManager (single public orchestration entry point)
         -> IntakeService: attachment validation, manifest, CAS
         -> WorkspaceService: per-task paths and path safety
         -> Scheduler: two active slots and FIFO
         -> Runner protocol
              -> ACPClient protocol (ACP v1 surface)
              -> MockRunner / MockACPClient (deterministic P0 implementation)
         -> SolverStore: authoritative SQLite writes and task events
         -> WriteupService: evidence-linked incremental Markdown
                    -> existing ArtifactStore / SHA-256 CAS

The current package is backend/app/solver/. Services are injected behind protocols so scheduling, storage, intake and Writeup can be tested independently.

## 2. Component ownership

| Concern | Owner | Must not be duplicated in |
|---|---|---|
| Task lifecycle/orchestration | TaskManager | UI, Worker prompt, ad-hoc SQL |
| Admission and FIFO | scheduler.py + Manager | independent queue JSON, frontend |
| Persistence/state transitions | SolverStore / existing SQLite event store | second JSONL or parallel DB writer |
| Worker lifecycle abstraction | Runner protocol | Manager-specific ACP calls |
| ACP wire surface | ACPClient adapter | category Skills |
| Per-task workspace | workspace.py | arbitrary Worker path construction |
| Attachment intake/integrity | intake.py + ArtifactStore | mutable copies of originals |
| Evidence bytes | content-addressed ArtifactStore | duplicate blob stores |
| Writeup rendering | writeup.py from persisted facts | free-form model reconstruction |
| Category knowledge | Core + category Skills (planned) | MCP server / persistence layer |
| Heavy fixed-version tools | dedicated tool adapter/MCP server (planned where justified) | prompt text or duplicate wrappers |
| Policy and OS isolation | execution boundary (planned gates) | Skill instructions alone |

## 3. Task and Worker model

- One task corresponds to one challenge and one Worker session at a time.
- Each task owns input/, work/, output/, scripts/ and writeup/ directories.
- Original attachments are immutable CAS objects; workspace copies are read-only and re-hashed.
- Manager persists task state, Worker identity/session references, event links and step metadata.
- Scheduler admits at most two active tasks. Queue order is persisted and FIFO.
- Paused is non-terminal and retains its slot in the current contract. Cancel is terminal and releases the slot only after confirmed shutdown.
- Manager restart turns live work into an explicit interrupted/recovery state; it must not pretend an in-memory Worker is alive.

## 4. Lifecycle and ACP

Persisted task states include queued, preparing, running, waiting_approval, paused, interrupted, solved, unsolved, failed and cancelled. Legal transitions live in backend/app/solver/models.py; do not bypass them with direct status mutation.

Worker shutdown is safety-critical: request stop/cancel, query and confirm execution is quiescent, then close the session, persist terminal state and release the slot. If stop is not confirmed, do not close as if successful or free the slot.

ACP v1 support is deliberately narrow. Unsupported ACP surfaces must raise an explicit unsupported error. Do not invent support for process kill, suspend/resume, terminal APIs or per-session preset controls.

## 5. Data and evidence flow

    Original ZIP / screenshot
      -> validate + hash
      -> immutable CAS
      -> read-only task input copy + manifest
      -> task events / Worker updates
      -> event references + CAS artifacts
      -> Writeup draft with evidence references and explicit gaps
      -> human review/edit/version
      -> final ZIP + manifest + SHA256SUMS

SQLite is the single authoritative metadata/event store. CAS is the authoritative byte store. Do not create a second authoritative JSONL event log. JSON/Markdown/ZIP are derived outputs, not alternate sources of truth.

Agent event payloads and candidate Flag plaintext must not be stored directly in SQLite. Store sensitive payload bytes in CAS and persist only required digest/reference and non-sensitive metadata. Export must disclose incomplete/missing evidence, tool/Worker state and Flag verification state.

## 6. Target product architecture — Scheme A

The authoritative target is **Skill as the brain, MCP as the hand, Cordis as the thin skeleton**, as captured in PRODUCT_BLUEPRINT.md and the 2026-11-10 feasibility audit.

- Harness Web GUI remains the one user-facing entry point. Manager Session asks for missing information, relays approvals, summarizes progress and presents results.
- Manager Core is the deterministic task/queue/state authority.
- Harness native subagents are the target Worker host under a dedicated ctf profile (after explicit user approval). One task gets one Worker session and one task-specific cwd.
- ctf-core plus one of five category Skills (Web/Misc/Crypto/Reverse/Pwn) supplies process and specialist knowledge.
- Harness native tools handle ordinary filesystem/shell/web operations. One custom CTF Tools MCP server handles justified heavy/fixed-version tools.
- Cordis is a thin host adapter: lifecycle bridge, permission relay, evidence tap and optional screenshot action. It does not run a second Agent loop or own a competing queue/database.
- The existing Workbench evidence backend remains authoritative for SQLite event/hash-chain records, CAS bytes, safe import, Writeup and export.
- WSL2/Docker is the later OS execution boundary for untrusted binaries and constrained network activity.

The feasibility audit considered four options. Scheme A is selected. Cordis platformization (B) is too expensive and host-version-sensitive for the first release; the Harness+Agoni dual-agent system (C) duplicates loops/context/evidence; MCP-max (D) cannot automatically observe every Harness tool event and would over-rely on Worker self-report.

### 6.1 How the P0 Manager Core fits

The accepted P0-REVIEW-FIX-02 code is an **engineering foundation**, not the whole product and not a replacement for the target topology.

P0 provides TaskManager, two-slot FIFO Scheduler, per-task workspace, intake, Runner/ACP contracts, deterministic Mock Runner, SQLite event persistence, CAS evidence and incremental Writeup. It deliberately does not provide a real ACP process, live Harness Worker, ctf profile, five category Skills, real binary execution, target access, Docker isolation or a separate product UI.

Integration guardrails:
- Manager Core remains the authoritative task/queue/state owner. Cordis bridges Host Worker lifecycle/events into it; Cordis must not maintain a second queue or state database.
- Runner remains the runtime seam. The real Host Worker adapter is an open A2 integration decision: compare the audited native subagent-control route with the P0 ACP adapter contract in an isolated smoke test. Do not claim either is connected until create/update/permission/stop/close/recovery are proven.
- ACP v1 support is deliberately narrow. Unsupported calls stay explicit failures; ACP cancellation is not an OS process kill.
- Reuse the existing SQLite/CAS backend. No parallel evidence store or JSONL authority.
- Keep Harness GUI as the product entry point. Do not expand the existing frontend into a second full-stack application.
- Passing Manager Core tests does not complete product P0; product P0 also requires the profile, initial Skills, evidence tap, real screenshot and end-to-end acceptance described in PRODUCT_BLUEPRINT.md.

## 7. Security boundary

A task-specific cwd is organizational isolation, not OS security isolation. A Runner/ACP adapter is a lifecycle abstraction, not a sandbox. Pwn/Reverse samples and untrusted binaries require a separate execution environment with filesystem, process, network and resource controls before real execution is enabled. See SECURITY.md.
