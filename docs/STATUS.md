# Current Project Status

Last reconciled: 2026-11-10 (Asia/Shanghai).

## Repository and review state

- Repository: Sunset-Sunrise-Sun/CTFworker.
- Default branch: main.
- Product/governance branch: docs/project-baseline.
- Initial product/architecture decision alignment commit: `15e652c90d11640e91156f021d4eaa86d1be1609`.
- A0-2 tracking issue: https://github.com/Sunset-Sunrise-Sun/CTFworker/issues/2
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
- Outer review package: `CTF-P0-review-fix-02-package.zip` (682,868 bytes; SHA-256 `53913ddb165a321284bb29bd84d2ea57dd30618889cf41e3d974b145274a403e`).
- Exact source ZIP member: `CTF-P0-review-fix-02-package/04-source-snapshot/ctf-manager-p0-fix-02-a4d0762-source.zip`.
- Inner ZIP SHA-256: `ef523c473ad1c047687191e10e93e7deed40db809e1670813b67a7d8f8b7c05f` (verified); ZIP comment/source label: `a4d0762e1b5b7673a09ab5a2ac99656bc743925d`.
- Inner archive: 110 files, 1,916,345 uncompressed bytes; no .git metadata. The ZIP comment is a provenance label, not Git ancestry proof. The earlier filename `ctfworker_p0_a4d0762_source.zip` was an inaccurate shorthand; use the exact nested member path above.

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

## Environment workstream — latest handoff

- Phase 0/1: reported complete. Ubuntu 24.04.5 WSL2 is installed at `D:\\WSL\\Ubuntu-24.04`; Linux user `ctfuser`.
- Phase 2A and Phase 2B-1A: reported complete.
- Docker Engine: `29.9.3`, installed and running inside WSL2.
- **Phase 2B-1G: PASS.** Docker systemd proxy drop-in points to the reachable WSL-side proxy `127.0.0.1:7890`; `hello-world` completed its first pull/run successfully. Docker socket is `root:docker` mode `0660`; `ctfuser` is not in the `docker` group; no TCP listeners on `2375/2376`; `.wslconfig` mirrored networking and `autoProxy` are effective.
- **Phase 2B-2A read-only audit and Phase 2B-2B-R1 design: complete, zero system changes.** R1 requires the Broker to be the sole Docker API trusted computing base; Worker receives neither Docker socket nor sudo; image references are digest-pinned; gVisor is fail-closed; default network is disabled; resource admission and file safety are enforced.
- **Next environment gate:** independent review of the 2B-2A/2B-2B-R1 evidence and design. Phase 2B-2C implementation is not authorized until that review passes and the user separately authorizes the specific implementation; any required Safety Net approval remains mandatory.
- This application branch does not authorize OS, firewall, proxy, Docker, security-setting or reboot changes.

## Current single next task

**A0-2 — import the exact original P0-REVIEW-FIX-02 source archive into docs/project-baseline.** Preserve the product north star, rename original source README/legacy Workbench architecture file, redact only the two documented local-path-bearing evidence docs, and run target-environment validation.

Do not start real ACP integration, Skills/MCP implementation or toolchain installation in the same task. Keep PR #1 draft/open/unmerged for independent review.
