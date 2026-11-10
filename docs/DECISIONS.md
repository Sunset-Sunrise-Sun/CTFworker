# Architecture Decision Log

Append decisions; do not silently rewrite history. When a later decision supersedes an earlier one, preserve the earlier entry and state the supersession explicitly.

## D-001 — Local-first, evidence-first product (accepted)
Writeups derive from recorded actions and evidence. A final Flag or model recollection is not a source of truth.

## D-002 — Manager/Worker, one task per Worker (accepted)
One challenge task owns one Worker session and one task workspace. Manager handles intake, scheduling, lifecycle, approval relay, recovery and reporting; Worker performs task-scoped analysis.

## D-003 — Two active slots, FIFO queue (accepted)
Default active challenge limit is two and hard limit is two. Additional tasks queue FIFO. Change only with resource measurements and explicit approval.

## D-004 — Runner and ACP adapter boundary (accepted)
Manager depends on a Runner protocol; ACP details are behind an ACPClient adapter. P0 uses a deterministic Mock Runner. Unsupported ACP surfaces fail explicitly.

## D-005 — SQLite is the only authoritative event/state writer (accepted)
Reuse the Workbench SQLite event store and CAS. No parallel JSONL source of truth or ad-hoc SQL writers. JSON/Markdown/ZIP are derived outputs.

## D-006 — Immutable originals and content-addressed artifacts (accepted)
Original attachments are retained immutably, hashed and copied read-only into task workspaces. Artifact bytes are CAS-addressed and verified by digest.

## D-007 — Evidence-backed Writeup (accepted)
Writeup is generated from persisted records, contains evidence references and marks gaps. Human-authored content is separately versioned.

## D-008 — Skill/MCP/Cordis are integration layers (accepted)
Skills hold solving knowledge and behavior; native tools handle ordinary operations; MCP wraps justified heavy/fixed-version tools; Cordis/Host integration provides lifecycle hooks, authorization relay and evidence taps. These layers do not replace Manager Core or the evidence backend.

## D-009 — No separate full-stack UI for first release (accepted)
Reuse the Harness GUI and existing Workbench UI/API. Add only necessary Solver surfaces; no parallel frontend without a new decision.

## D-010 — Mock success is not live execution (accepted)
Mock Runner, synthetic challenge, live ACP session, real binary execution, authorized target interaction and platform Flag verification are separate evidence classes.

## D-011 — Public repository contains no real challenge data (accepted)
Only code, generic docs, synthetic fixtures and redacted evidence may be committed.

## D-012 — P0-REVIEW-FIX-02 is the accepted code baseline (2026-11-10)
Source label: `a4d0762e1b5b7673a09ab5a2ac99656bc743925d`. Preserve its behavior and frozen contracts during repository import. Migration changes must be isolated and independently reviewed.

## D-013 — Environment setup is a separate workstream (accepted)
WSL2/Docker/proxy setup does not change Manager Core contracts. Host setup and application development have separate tasks, reports and approvals. Check the latest environment handoff before any environment action.

## D-014 — Scheme A is the target product architecture (accepted; Worker-host detail superseded by D-018)
Keep one Harness entry point; Skills for knowledge; native tools for ordinary work; one heavy-tool MCP; thin Cordis/Host hooks; existing evidence backend. The early native-subagent Worker proposal is superseded by D-018.

## D-015 — Manager Core is the single task/queue authority (accepted)
Manager Core owns persisted task state, two-slot FIFO admission, lifecycle and task-to-evidence links. Cordis bridges host events/lifecycle/permissions and must not maintain a competing queue or database.

## D-016 — Native-subagent-vs-ACP Worker spike (superseded by D-018)
The earlier proposal left runtime choice open. The subsequent workspace-root audit established that subagent cwd does not change the session writable root. The accepted Worker carrier is now recorded in D-018.

## D-017 — Product P0–P3 differs from engineering P0 Manager Core (accepted)
Product P0 includes environment/profile preparation, Skills, evidence hooks, screenshots and end-to-end proof. Passing engineering P0 Manager Core tests does not complete Product P0.

## D-018 — One independent ACP Worker Session per task (accepted, 2026-11-10)
Each task gets its own ACP session created with `workspace=tasks/<task_id>/work`. The session workspace is the relevant write root; a child Agent cwd is not a substitute. ACP v1 is limited to the verified session methods in PRODUCT_BLUEPRINT.md and ARCHITECTURE.md. ACP process restoration is not claimed; task recovery is supported. Manager/Runner/wrapper supplies functions ACP does not expose. No real ACP integration is claimed until A2 smoke tests pass.

## D-019 — CLI/headless-first solving (accepted, 2026-11-10)
Workers solve through command-line and headless tools. GUI launch and screenshot collection are supported, but autonomous clicking of all GUI controls is not a first-release requirement.

## D-020 — G0 first-start environment readiness (accepted, 2026-11-10)
Before formal solving, run a doctor for the baseline toolchain, install/configure missing tools through the approved environment workflow, run self-tests and freeze versions in `toolchain.lock`. Workers do not install tools or modify the Windows host during a solve. Warm-up challenges may extend the reviewed tool inventory.

## D-021 — Explore autonomously before escalation (accepted, 2026-11-10)
Worker checks installed tools, tries appropriate alternatives and performs reasonable local exploration before asking the user. It reports only when genuinely blocked or when a human decision/authorization/input is required. No arbitrary attempt loop or unsafe brute force.

## D-022 — Category-specific solving strategies (accepted, 2026-11-10)
- Misc encrypted archives: analyze format/metadata/password hints, then bounded local attempts; escalate only after reasonable paths are exhausted.
- Crypto: mathematical analysis first, combined with reproducible tool calculation.
- Reverse: static analysis first; dynamic execution only in an accepted independent isolation environment.
- Pwn: local analysis/testing first; remote exploitation requires a one-time task/target/time-window grant.
- Web: source/configuration/local reproduction first; remote requests only to explicitly authorized challenge targets.

## D-023 — Pwn authorization uses bounded window semantics (accepted, 2026-11-10)
One approval permits a continuous sequence for the same task and target within the approved time window and action scope. Completion or expiry invalidates it. It never transfers across tasks or targets. Each protected operation still checks the grant.

## D-024 — Separate auxiliary research and challenge-range channels (accepted, 2026-11-10)
Generic web research is separate from challenge-range access. Target hosts are blocked from the auxiliary channel; challenge traffic uses structured `ctf-range-*` tools with per-call scope and grant checks. No generic-web or arbitrary-shell bypass.

## D-025 — G0 is not ad-hoc tool installation (accepted, 2026-11-10)
The baseline toolchain is prepared before formal solving. Unexpected missing tools during a task trigger alternative checks and an environment blocker report, not Worker-side installation. New tools found during warm-up are added through a reviewed toolchain update.

## D-026 — P0 source import is a migration, not a rewrite (accepted)
Import the original P0-REVIEW-FIX-02 source snapshot unchanged except for the two documented public-path redactions and file renames needed to preserve product docs. No runtime refactor or phase expansion in A0-2.
