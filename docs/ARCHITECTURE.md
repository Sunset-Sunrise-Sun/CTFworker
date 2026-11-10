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

## 6. Historical integration proposal (not current implementation)

The earlier feasibility study proposed **Skill as solving playbook, MCP as heavy-tool interface, Cordis as thin scheduling/authorization/evidence-hook layer, and Harness-native tools for ordinary file and shell work**. It also recommended a dedicated CTF profile, category-specific Worker presets, and event-tap evidence capture.

This remains an integration direction, not a claim those Harness/Cordis pieces are implemented in this repository. The later P0 implementation deliberately established a portable Manager/Runner/ACP contract first.

Retained principles: use native tools for ordinary operations where appropriate; use MCP only for fixed-version/heavy structured tools; never let MCP bypass authorization and isolation. Skills provide instructions and domain knowledge, not security enforcement.

## 7. Security boundary

A task-specific cwd is organizational isolation, not OS security isolation. A Runner/ACP adapter is a lifecycle abstraction, not a sandbox. Pwn/Reverse samples and untrusted binaries require a separate execution environment with filesystem, process, network and resource controls before real execution is enabled. See SECURITY.md.
