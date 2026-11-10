# A0-2 — Import the exact P0-REVIEW-FIX-02 source snapshot

**Task ID:** A0-2  
**Phase:** Engineering A0 / P0 source migration  
**Target branch:** docs/project-baseline (the head branch of Draft PR #1)  
**Source archive:** original ctfworker_p0_a4d0762_source.zip  
**Expected archive SHA-256:** ef523c473ad1c047687191e10e93e7deed40db809e1670813b67a7d8f8b7c05f  
**Source label in ZIP comment:** a4d0762e1b5b7673a09ab5a2ac99656bc743925d

## Product outcome

Import the accepted P0 Manager Core code into CTFworker without changing its runtime behavior, while retaining the product north-star docs already committed on this branch. The result must be a reviewable migration candidate, not a claim that the full CTF Solver product is complete.

## Mandatory pre-read

Read:
- README.md
- docs/PRODUCT_BLUEPRINT.md
- docs/PRODUCT_CHARTER.md
- docs/ARCHITECTURE.md
- docs/ROADMAP.md
- docs/STATUS.md
- docs/DECISIONS.md
- docs/SECURITY.md
- docs/MIGRATION_AUDIT_20261110.md
- docs/P0_SOLVER_MANAGER.md and docs/P0_SOLVER_INTERFACES.md after import

The product blueprint is authoritative. Do not alter its product scope, Scheme A target, five categories, one-task/one-Worker rule, two-worker FIFO, single evidence authority, approval boundary or Product P0–P3 gates.

## Source selection and integrity

1. Use the original 110-file source archive with the expected SHA-256 above. Verify it before extraction.
2. Do not use the later 119-file convenience migration bundle as the source snapshot; it contains a replaced README and replaced docs/ARCHITECTURE.md.
3. The source archive has no .git directory. Its ZIP comment labels the source SHA; do not claim that archive metadata alone proves Git ancestry.
4. Do not substitute dd299d9 or an earlier review package.
5. Do not rewrite historical SHA manifests to make mismatches disappear. Record the known manifest status in docs/MIGRATION_AUDIT_20261110.md.

## Import mapping — preserve current north-star files

Extract to a temporary directory, then copy into the repository root while preserving .git.

Copy all source files except these two paths:
- Source root README.md -> target README-P0-ORIGINAL.md. Do not overwrite the new target README.md.
- Source docs/ARCHITECTURE.md -> target docs/reference/WORKBENCH_ARCHITECTURE.md. Do not overwrite the new target docs/ARCHITECTURE.md.

All other source files should retain their original relative paths unless the redaction rule below applies. This includes backend/, frontend/, scripts/, tests/, .github/workflows/ci.yml, .gitignore and the P0 interface/spec/test evidence docs.

The current target branch's governance files must remain unchanged:
- README.md
- MIGRATION_GUIDE.md
- docs/PRODUCT_BLUEPRINT.md
- docs/PRODUCT_CHARTER.md
- docs/ARCHITECTURE.md
- docs/ROADMAP.md
- docs/STATUS.md
- docs/AGENT_WORKFLOW.md
- docs/DECISIONS.md
- docs/SECURITY.md
- docs/MIGRATION_AUDIT_20261110.md

Add root P0_SOURCE_PROVENANCE.md recording source archive name/hash, ZIP comment source label, file count, uncompressed size, and the fact that .git metadata is absent.

## Required public-repository redaction

The original source contains local absolute Windows paths in:
- docs/a00/REGRESSION.md
- docs/MVP_WRITEUP_ACCEPTANCE.md

Replace only the local path strings with neutral placeholders such as <LOCAL_WORKSPACE_PATH> and <TEMP_WORKSPACE_PATH>. Preserve command structure, test results and surrounding evidence. Do not publish local usernames, machine paths, environment dumps, challenge data, real Flags, credentials or tokens.

Synthetic Flag strings in tests and solver_demo.py are deliberate synthetic fixtures for the redaction tests. Keep them unchanged.

## Exact non-goals

- No runtime code changes, refactors, formatting churn or dependency upgrades.
- No real ACP session, live model provider, real challenge binary or target network execution.
- No Docker/WSL/firewall/proxy/VBS/HVCI/profile/security-setting changes.
- No new queue, database, JSONL event store, frontend or second Agent loop.
- No merge, release, auto-merge, force-push or history rewrite.

## Validation commands

Run on the user's Windows development environment with the project's pinned dependencies. Capture exact command, exit code and output.

Backend:
    cd backend
    py -m pip install -e ".[dev]"
    py -m ruff check app tests
    py -m pytest -q
    cd ..

Offline synthetic demo:
    py scripts/solver_demo.py

Frontend:
    cd frontend
    npm ci
    npm run build
    cd ..

Also verify:
- original attachment immutability and hash checks;
- two active slots + FIFO ordering;
- pause retains slot and cancel is terminal;
- Worker stop confirmation precedes session close and slot release;
- CAS/event links and no plaintext Flag/agent payload in SQLite;
- restart recovery and no second JSONL authority;
- no local paths/secrets/real challenge artifacts in git diff;
- no __pycache__, .pytest_cache, .ruff_cache, node_modules, dist, sqlite/db, .env or generated challenge data staged.

## Known environment-specific test/CI issue

The current workflow uses ubuntu-latest, but test_solver_review_regressions.py::test_08_junction_reparse_cross_task_access_rejected calls Windows cmd /c mklink /J unconditionally. Linux execution raises FileNotFoundError. Do not silently alter the P0 import commit to hide this. Report it as a separate test-portability follow-up; the Windows junction test must still be exercised on Windows. A separate focused fix may be proposed after this exact import is reviewed.

## Required output

1. Starting branch and exact HEAD.
2. Source archive SHA verification output.
3. Import mapping and changed-file count.
4. Exact paths redacted and confirmation no unrelated data was changed.
5. P0 manifest verification result and known mismatches (do not rewrite manifests).
6. Full backend, Ruff, demo and frontend results with skips/failures.
7. CI portability issue disposition.
8. Exact commit SHA and PR #1 status. Keep PR draft/open/unmerged for independent review.
9. Update docs/STATUS.md with migration state and the next single task.

**Stop condition:** if the archive hash differs, protected north-star docs would be overwritten, a real secret/challenge artifact is found, the tests expose a code regression, or the working tree contains unrelated changes, stop and report before proceeding.
