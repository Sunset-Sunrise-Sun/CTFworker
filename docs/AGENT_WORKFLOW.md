# Agent Workflow — Mandatory Handoff Contract

This is the anti-drift contract for every new ChatGPT conversation, DeepSeek session, Codex task and human contributor.

## 1. Mandatory startup sequence

Before planning or editing code:

1. Read README.md.
2. Read docs/PRODUCT_BLUEPRINT.md first. It is the product north star.
3. Read docs/PRODUCT_CHARTER.md.
4. Read docs/ARCHITECTURE.md; distinguish current P0 implementation from target Harness integration.
5. Read docs/ROADMAP.md; distinguish Product P0–P3, Engineering A0–A7 and Environment B0–B5.
6. Read docs/STATUS.md and work only on its single next task unless the user explicitly assigns another.
7. Read docs/DECISIONS.md and docs/SECURITY.md.
8. Read the current task specification and frozen interfaces.
9. Inspect the actual GitHub default branch, target branch, open PRs, latest commit and local working tree. Never trust a handoff SHA without rechecking it.

If documents conflict, stop and report the conflict. The selected architecture is Scheme A with one independent ACP Worker Session per task. Do not silently revert to a native subagent cwd model, generic chat wrapper, second full-stack UI, second Agent loop, duplicate queue/database or MCP-max architecture.

## 2. Single-task discipline

- Work only on the task named in docs/STATUS.md or explicitly assigned by the user.
- One task, one branch, one reviewable change. No opportunistic refactors.
- Preserve one task/one ACP Worker/one workspace, two active slots + FIFO, G0 readiness, CLI/headless-first solving, human approval, dual network channels, one SQLite/CAS authority and evidence-derived Writeups.
- Manager owns task state/queue/lifecycle; Cordis is a thin Host bridge.
- Worker explores safe installed options before escalating. Worker never installs host software.
- Skills teach solving; MCP exposes structured heavy tools and must self-enforce; OS/container controls provide hard isolation.
- Do not claim Product P0 complete merely because engineering P0 Manager Core tests pass.
- If new scope is discovered, record it as follow-up and ask before expanding.

## 3. Required task brief

Every implementation task states task ID/phase, product outcome, in-scope behavior, non-goals, interfaces to preserve, expected files, safety/permission constraints, tests/acceptance evidence, stop conditions and rollback plan.

## 4. Required implementation report

Report starting branch/exact SHA, changed files and diff summary, behavior changed/unchanged, actual commands and exit codes, test counts and skip reasons, security/failure-path coverage, artifacts/checksums, known limitations/deferred work, and commit/PR state.

Never accept "tests pass" without command and output. Never claim an unrun test. Separate historical test evidence from tests rerun in the current repository/environment.

## 5. Independent review

Reviewer inspects code and tests, not only implementer report. At minimum inspect:
- frozen interfaces and state transitions;
- persistence transaction boundaries;
- stop/cancel/close ordering and slot release;
- traversal, symlink, junction and reparse-point handling;
- ZIP validation and immutable-original guarantees;
- CAS reuse and sensitive payload handling;
- Flag state and plaintext handling;
- Writeup provenance, missing-evidence disclosure and export snapshot consistency;
- ACP session workspace root and recovery semantics;
- MCP task/permission/network enforcement;
- G0 toolchain doctor and version lock;
- tests in the actual migration repository.

Review outcomes: PASS, PASS-WITH-NOTES, REQUEST-CHANGES, BLOCKED. Track notes.

## 6. Git and environment authorization

Unless the user explicitly authorizes otherwise:
- use a feature branch;
- do not push, create PR, mark ready, merge, enable auto-merge, tag release or rewrite history;
- do not alter unrelated branches or the old repository;
- never force-push;
- never delete user changes or reset a dirty worktree.

Environment actions are separate from source-code permission. Do not change OS features, WSL2, Docker, firewall, proxy, security controls, administrator settings or reboot without explicit user authorization and any required Safety Net policy approval.

## 7. Context compaction

At the end of each task:
- update docs/STATUS.md with exact phase, branch and HEAD;
- append to docs/DECISIONS.md if a decision changed;
- preserve detailed history in versioned reports;
- state one next task and exact resume instructions.

The repository is the source of truth; do not rely on a long chat history.
