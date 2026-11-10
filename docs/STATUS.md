# Current Project Status

Last reconciled: 2026-11-10 (Asia/Shanghai).

## Repository and review state

- Repository: Sunset-Sunrise-Sun/CTFworker.
- Default branch: main.
- Product/governance branch: docs/project-baseline.
- Current branch HEAD: `15e652c90d11640e91156f021d4eaa86d1be1609` (product/architecture decision alignment commit).
- Draft PR #1: https://github.com/Sunset-Sunrise-Sun/CTFworker/pull/1
- PR #1 is open, draft and unmerged. It contains the product framework, architecture, roadmap, decisions, security policy, migration audit and A0-2 task specification.
- P0 source code has **not yet been imported into GitHub**. This branch is a product/documentation baseline, not yet a complete runnable migration.

## Product north star — frozen

docs/PRODUCT_BLUEPRINT.md is authoritative:
- one Harness entry point and Manager/Worker model;
- Scheme A: Skills for knowledge, native tools for ordinary work, one heavy-tool CTF MCP, thin Cordis/Host hooks, existing Workbench evidence backend;
- **one independent ACP Worker Session per task**, created with `workspace=tasks/<task_id>/work`;
- one task/one Worker/one workspace;
- two active tasks maximum, third and later FIFO;
- CLI/headless first; GUI screenshots supported, autonomous clicking not required;
- G0 first-start doctor, baseline toolchain preparation and `toolchain.lock`;
- autonomous exploration before asking the user;
- Misc bounded encrypted-archive analysis; Crypto math + tool calculation; Reverse static-first and isolated dynamic; Pwn local-first and one-time scoped remote grant; Web source/local-first and exact authorized remote scope;
- human approval, dual network channels, single SQLite/CAS authority and evidence-derived Writeup;
- Product P0–P3 gates.

The earlier native-subagent Worker proposal and "runtime choice open" text are superseded by D-018. Do not use native subagent cwd as the Worker write-root boundary.

## Accepted engineering source baseline

- P0-REVIEW-FIX-02 source label: `a4d0762e1b5b7673a09ab5a2ac99656bc743925d`.
- Original archive: `ctfworker_p0_a4d0762_source.zip`.
- Expected SHA-256: `ef523c473ad1c047687191e10e93e7deed40db809e1670813b67a7d8f8b7c05f`.
- Archive: 110 files, 1,916,345 uncompressed bytes; no .git metadata. ZIP comment is a source label, not Git ancestry proof.

## A0-2 migration audit

See docs/MIGRATION_AUDIT_20261110.md and docs/tasks/A0-2_P0_SOURCE_IMPORT.md.

- Archive SHA verified locally against the expected value.
- P0 solver manifest: 24 raw matches + 6 LF-normalized matches; two documentation mismatches (README.md and docs/P0_SOLVER_FILES.md). Do not rewrite historical manifests.
- KAT manifest: 4/4 match after LF normalization.
- Two historical docs contain local absolute Windows paths and require targeted redaction before public commit.
- Offline demo: 58/58 passed.
- Backend non-root Linux run excluding the Windows-only junction test: 536 passed, 2 skipped, 1 deselected.
- Root Linux run: 533 passed, 2 skipped, 4 failed (three permission tests behave differently under root; one Windows junction test requires cmd.exe).
- Ruff unavailable in audit container; frontend npm ci --offline blocked because vue-tsc was not cached.
- Ubuntu CI invokes a Windows-only junction test unconditionally. Track a separate focused portability fix; do not hide it in the exact source import.

These are historical/local audit results, not target-repository acceptance. Full pinned Windows/Ruff/frontend validation remains required.

## Known P0 follow-ups

1. CAS reuse can preserve the first Artifact's evidence_kind when identical bytes are referenced later by a different semantic source; export/de-identification must inspect all references/provenance.
2. A stale Worker lifecycle comment says cancelled where current status is closed; update with a focused change and tests.
3. Preserve historical checksum manifests and disclose known documentation/line-ending mismatches.
4. Rerun the complete suite in the actual migration repository and CI; do not copy old counts as current acceptance.

## Environment workstream

- B0 Windows baseline audit: reported complete.
- B1 WSL2 networking: reported PASS.
- B2 dockerd proxy inheritance: outstanding in last environment handoff.
- B3 Docker Engine validation: not confirmed complete.
- A narrowly scoped CC Safety Net exception package was prepared for Docker's public APT key. Confirm actual formal approval state with the environment owner/admin before any blocked action.
- This application branch does not authorize OS, firewall, proxy, Docker, security-setting or reboot changes.

## Current single next task

**A0-2 — import the exact original P0-REVIEW-FIX-02 source archive into docs/project-baseline.** Preserve the product north star, rename original source README/legacy Workbench architecture file, redact only the two documented local-path-bearing evidence docs, and run target-environment validation.

Do not start real ACP integration, Skills/MCP implementation or toolchain installation in the same task. Keep PR #1 draft/open/unmerged for independent review.
