# Current Project Status

Last reconciled: 2026-11-10 (Asia/Shanghai).

## Repository and review state

- Repository: Sunset-Sunrise-Sun/CTFworker.
- Default branch: main.
- Product/governance baseline branch: docs/project-baseline.
- Draft PR #1: https://github.com/Sunset-Sunrise-Sun/CTFworker/pull/1
- PR #1 is open, draft and unmerged. It currently contains the product north star, architecture, roadmap, decisions, security, migration audit and A0-2 task specification.
- **The P0 source code has not yet been imported into GitHub.** The docs branch is not a complete code migration.

## Product north star — frozen

docs/PRODUCT_BLUEPRINT.md is the authoritative product framework:
- one Harness GUI / Manager Session;
- Scheme A: Skills for solving knowledge, native Harness tools for ordinary work, one CTF Tools MCP for heavy tools, thin Cordis host hooks, existing Workbench evidence backend;
- five categories: Web, Misc, Crypto, Reverse and Pwn;
- one task = one Worker = one task workspace;
- at most two active Workers, FIFO beyond capacity;
- human approval for sensitive actions;
- SQLite + CAS as the single evidence authority;
- evidence-derived Writeup and verifiable ZIP export;
- Product P0–P3 acceptance gates.

Do not confuse **Product P0** (profile + initial Skills + evidence service/tap + screenshot + doctor + E2E proof) with the accepted **engineering P0 Manager Core** source milestone.

## Accepted engineering source baseline

- P0-REVIEW-FIX-02 source label: a4d0762e1b5b7673a09ab5a2ac99656bc743925d.
- Original source archive: ctfworker_p0_a4d0762_source.zip.
- Archive SHA-256: ef523c473ad1c047687191e10e93e7deed40db809e1670813b67a7d8f8b7c05f.
- Original archive: 110 files, 1,916,345 uncompressed bytes; no .git metadata.
- Use the original source archive, not the later 119-file convenience bundle, which replaced README.md and docs/ARCHITECTURE.md.

## A0-2 migration audit results

Detailed evidence: docs/MIGRATION_AUDIT_20261110.md. Exact Codex task: docs/tasks/A0-2_P0_SOURCE_IMPORT.md.

- Source archive SHA and ZIP comment verified.
- P0 solver manifest: 24 raw matches + 6 LF-normalized matches; two documentation mismatches (README.md and docs/P0_SOLVER_FILES.md). Backend/app, backend/tests and solver_demo entries match after accounting for line endings.
- KAT manifest: 4/4 match after LF normalization.
- Two historical docs contain local absolute paths and must be redacted before public commit.
- Offline demo: **58/58 passed**.
- Backend tests as non-root Linux user, excluding only the Windows-only junction test: **536 passed, 2 skipped, 1 deselected**.
- Full root Linux run: **533 passed, 2 skipped, 4 failed**; three read-only tests fail because root bypasses Unix mode bits, and one Windows-only junction test cannot find cmd.exe.
- Ruff not available in this container; frontend npm ci --offline blocked because vue-tsc was not cached.
- The supplied Ubuntu CI workflow invokes a Windows-only junction test unconditionally. Track a separate focused test-portability fix; do not silently alter the exact P0 import.

These are local migration-audit results, not target-repository CI or Windows acceptance. Full pinned Windows/Ruff/frontend validation remains required.

## Known P0 follow-up notes (not automatic blockers)

1. CAS reuse may retain the first Artifact's evidence_kind when identical bytes are referenced later by a different semantic source. Export/de-identification must inspect all references/provenance.
2. A stale Worker lifecycle comment says cancelled where current status is closed; update only when touching that code, with focused tests/docs.
3. Preserve the historical SHA manifests; do not rewrite them to conceal line-ending or documentation mismatches.

## Host environment workstream

- Phase 0 Windows baseline audit: reported complete.
- Phase 2B-1F mirrored networking acceptance: reported PASS.
- Outstanding: **dockerd did not inherit the host/WSL proxy**.
- A narrowly scoped CC Safety Net exception for Docker's public APT key was pending in the last recorded report. Verify current status before any blocked action.
- No changes to .wslconfig, firewall, proxy, VBS/HVCI, Docker daemon, OS features or reboot without explicit approval.

## Current single next task

**A0-2: import the original 110-file P0 source archive into the docs/project-baseline branch, preserve the product north star, redact local paths, and run pinned Windows validation.**

Do not start real ACP integration, Skills/MCP implementation or toolchain installation in the same change. Keep PR #1 draft/open/unmerged for independent review.
