# Current Project Status

Last reconciled: 2026-11-10 (project clock Asia/Shanghai).

## Repository

- New repository: Sunset-Sunrise-Sun/CTFworker.
- Default branch: main; repository was empty at initialization.
- Documentation branch: docs/project-baseline.
- The README initialization commit is on main. Product-baseline docs are being added on the documentation branch.
- **The P0 source snapshot has not yet been imported into this GitHub repository.** The documentation branch is not a complete code migration.

## Product/code baseline

- Accepted source baseline: **a4d0762e1b5b7673a09ab5a2ac99656bc743925d** (P0-REVIEW-FIX-02).
- The supplied source archive contains the P0 Manager Core implementation and its docs/tests.
- P0 scope: task intake, immutable attachment handling, per-task workspace, two-slot FIFO scheduler, persisted task/Worker states and recovery, ACP v1 adapter contract, deterministic Mock Runner, SQLite event persistence, CAS artifacts and incremental Writeup.
- P0 was independently accepted in the prior review, with follow-up notes. Do not reopen P0 broadly without a concrete regression.

## P0-REVIEW-FIX-02 acceptance snapshot

Reported historical candidate evidence:
- Full backend: **532 passed, 7 skipped**.
- Demo: **58/58**.
- New review-fix regression set: **26 passed**.
- Ruff: passed.
- Post-commit full pytest and Ruff: passed.

These are source-candidate results, not yet a fresh run from this new repository. Keep all 7 skips disclosed; platform/environment-limited tests are not silently counted as passed. Re-run checks after source import.

## Follow-up notes (not P0 blockers)

1. CAS reuse may retain the first Artifact's evidence_kind when identical bytes are referenced later by a different semantic source. Export/de-identification must inspect all references/provenance, not rely on the first Artifact kind alone.
2. A stale Worker lifecycle comment uses cancelled where current status is closed; update only when touching that code, with focused tests/docs.
3. The source SHA manifest has line-ending-sensitive mismatches for several files in one review package. Verify raw bytes and normalization policy before changing it. Never rewrite checksums to hide a mismatch.

## Host environment workstream

- Phase 0 Windows baseline audit: reported complete.
- Phase 2B-1F mirrored networking acceptance: reported PASS.
- Outstanding at handoff: **dockerd did not inherit the host/WSL proxy**. This is an environment task, not a reason to alter Manager Core.
- A narrowly scoped CC Safety Net exception request for writing Docker's public APT repository key was pending in the last recorded report. Do not assume approval or execute the blocked action without checking current status.
- Do not change .wslconfig, firewall, proxy, VBS/HVCI, Docker daemon, OS features or reboot without explicit approval.

## Current single next task

**A0-2: import the exact P0-REVIEW-FIX-02 source snapshot into this repository and run migration verification.**

Do not begin real ACP integration or toolchain work in the same change. Preserve the P0 snapshot; link product docs without changing runtime behavior.
