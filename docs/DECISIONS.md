# Architecture Decision Log

Append new decisions. Do not silently rewrite a prior decision; supersede it with a dated entry and rationale.

## D-001 — Local-first, evidence-first product (accepted)
CTFworker is a local-first CTF workbench whose Writeup derives from recorded actions and evidence. The final Flag or model recollection is not a source of truth.

## D-002 — Manager/Worker, one task per Worker (accepted)
One challenge task owns one Worker session and one workspace. Manager handles intake, scheduling, lifecycle, approval relay and reporting; Worker performs task-scoped analysis.

## D-003 — Two active slots, FIFO queue (accepted)
The default admission limit is two active challenge tasks; remaining tasks queue FIFO. Change only with resource measurements and an explicit decision.

## D-004 — Runner and ACP adapter boundary (accepted)
Manager depends on a Runner protocol; ACP details are behind an ACPClient adapter. P0 uses a deterministic Mock Runner. Unsupported ACP surfaces fail explicitly. Manager must not import concrete runtime details.

## D-005 — SQLite is the only authoritative event/state writer (accepted)
Reuse the Workbench SQLite event store and CAS. No parallel JSONL source of truth or ad-hoc SQL writers. JSON/Markdown/ZIP are derived outputs.

## D-006 — Immutable originals and content-addressed artifacts (accepted)
Original attachments are retained immutably, hashed, and copied read-only into task workspaces. Artifact bytes are CAS-addressed and verified by digest.

## D-007 — Evidence-backed Writeup (accepted)
Writeup is generated from persisted records, contains evidence references, and marks gaps. Human-authored content is separately versioned and cannot be confused with generated claims.

## D-008 — Skill/MCP/Cordis are integration layers, not current shipped claims (accepted)
The feasibility study's conceptual split remains: Skills hold solving knowledge and behavior; native tools handle ordinary operations; MCP wraps justified heavy/fixed-version tools; Cordis/host integration may provide scheduling hooks, authorization and evidence taps. The P0 implementation is the portable Manager/Runner/ACP core, not a completed Harness/Cordis plugin suite.

## D-009 — No separate full-stack UI for first release (accepted)
Reuse the existing Workbench UI/API. Add only necessary Solver surfaces; do not build a parallel frontend without a new decision.

## D-010 — Mock success is not live execution (accepted)
Mock Runner, mock provider, synthetic challenge, live ACP session, real binary execution, authorized target interaction and platform Flag verification are distinct stages and must be reported separately.

## D-011 — Public repository contains no real challenge data (accepted)
Only code, docs, synthetic fixtures and redacted evidence may be committed.

## D-012 — P0-REVIEW-FIX-02 is the accepted code baseline (2026-11-10)
Baseline source commit: a4d0762e1b5b7673a09ab5a2ac99656bc743925d. Preserve its behavior and frozen contracts during repository import. Any migration change must be isolated, justified and independently reviewed.

## D-013 — Environment setup is a separate workstream (2026-11-10)
WSL2/Docker/proxy setup does not change the Manager Core contract. Host setup and application development have separate tasks, reports and approvals. Latest known environment blocker: dockerd proxy inheritance after Phase 2B-1F; verify current state before resuming.

## D-014 — Scheme A is the target product architecture (2026-11-10)
The target product remains Harness-native single-window Manager + native per-task subagents under a dedicated ctf profile + Core/five category Skills + one heavy-tool CTF MCP + thin Cordis host hooks + reused Workbench evidence backend. Cordis is not a second Agent platform. The detailed responsibilities and product acceptance gates are in PRODUCT_BLUEPRINT.md.

## D-015 — Manager Core is the single task/queue authority (2026-11-10)
The later P0 Manager Core implementation owns persisted task state, two-slot FIFO admission, lifecycle and task-to-evidence links. The initial feasibility audit assigned a thin scheduler role to Cordis; after Manager Core was implemented, that responsibility is centralized in Manager Core to avoid two queues or competing stores. Cordis only bridges Host events/lifecycle/permissions and evidence hooks.

## D-016 — Real Worker host binding remains an explicit A2 spike (2026-11-10)
The feasibility audit recommended native Harness subagent-control under the dedicated ctf profile. P0 separately defines a Runner/ACP adapter contract and a Mock Runner. These are not assumed equivalent. A2 must compare the native subagent route and ACP adapter against required create/update/permission/stop/close/recovery semantics and record a decision before implementation. Until then, no real Worker integration is claimed.

## D-017 — Product P0–P3 and engineering P0 Manager Core are different tracks (2026-11-10)
Product P0 means profile + initial Skills + evidence service/tap + real screenshot + doctor + end-to-end acceptance. P0-REVIEW-FIX-02 is an engineering code milestone. Passing the latter does not mark the former complete.
