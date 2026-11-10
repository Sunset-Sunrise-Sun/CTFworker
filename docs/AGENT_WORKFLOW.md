# Agent Workflow — Mandatory Handoff Contract

This is the anti-drift contract for every new ChatGPT conversation, DeepSeek session, Codex task or human contributor.

## 1. Mandatory startup sequence

Before proposing or editing code:

1. Read README.md.
2. Read docs/PRODUCT_BLUEPRINT.md first. It is the product north star and the source for target architecture and product P0–P3 acceptance.
3. Read docs/PRODUCT_CHARTER.md.
4. Read docs/ARCHITECTURE.md, including the distinction between target Harness architecture and the P0 Manager Core implementation.
5. Read docs/ROADMAP.md; distinguish product P0–P3 from engineering milestones.
6. Read docs/STATUS.md.
7. Read docs/DECISIONS.md.
8. Read docs/SECURITY.md.
9. Read the current phase specification and relevant interface docs.
10. Inspect the actual GitHub default branch, open PRs, working tree and latest commit. Never trust a handoff SHA without rechecking it.

If documents conflict, stop and report the conflict. Do not choose the easiest interpretation. Never replace the selected Scheme A target with a generic chat wrapper, a second full-stack UI, a second Agent loop, a duplicate queue/database or MCP-max architecture without an explicit user-approved ADR.

## 2. Single-task discipline

- Work only on the task named in docs/STATUS.md or explicitly assigned by the user.
- One task, one branch, one reviewable change. No opportunistic refactors.
- Do not skip dependencies, acceptance gates or failed tests.
- Do not implement later-phase features as "small improvements".
- Preserve one task/one Worker/one workspace, two active Workers + FIFO, a single SQLite/CAS evidence authority, human approval, and evidence-derived Writeups.
- Treat Skills as knowledge/instructions, MCP as structured heavy-tool execution, Cordis as thin host hooks, Manager Core as task/queue authority, and OS/container controls as the actual isolation boundary.
- Do not claim product P0 complete merely because engineering P0 Manager Core tests pass.
- If new scope is discovered, record it as follow-up and ask before expanding.

## 3. Required task brief

Every implementation task must state task ID/phase, product outcome, in-scope behavior, non-goals, interfaces to preserve, expected files, safety/permission constraints, tests/acceptance evidence, stop conditions and rollback plan.

## 4. Required implementation report

Report starting branch/SHA, exact changed files and diff summary, behavior changed/unchanged, actual commands and exit codes, test counts and skip reasons, security/failure-path coverage, artifacts/checksums, known limitations/deferred work, and commit/PR state.

Never accept "tests pass" without the command and output. Never claim a test that was not run.

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
- tests in the actual migration repository.

Review outcomes: PASS, PASS-WITH-NOTES, REQUEST-CHANGES, BLOCKED. Track notes.

## 6. Git authorization

Unless the user explicitly authorizes otherwise:
- use a feature branch;
- do not push, create PR, mark ready, merge, enable auto-merge, tag release or rewrite history;
- do not alter unrelated branches or the old repository;
- never force-push;
- never delete user changes or reset a dirty worktree.

## 7. Context compaction

At the end of each task, update docs/STATUS.md and append to docs/DECISIONS.md if a decision changed. Handoff must include current phase/task, branch and exact HEAD, completed acceptance evidence, blockers, next single task and resume commands.

Keep STATUS.md short/current; preserve detailed history in versioned docs. The next agent must not infer the mainline from chat history.
