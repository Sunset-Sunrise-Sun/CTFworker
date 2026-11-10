# P0-REVIEW-FIX-02 Migration Audit — 2026-11-10

## 1. Source artifact identity

The source-of-truth input for migration is the original source archive, not the later convenience migration bundle.

| Item | Verified value |
|---|---|
| Source snapshot label in ZIP comment | a4d0762e1b5b7673a09ab5a2ac99656bc743925d |
| Source archive | ctfworker_p0_a4d0762_source.zip |
| Archive SHA-256 | ef523c473ad1c047687191e10e93e7deed40db809e1670813b67a7d8f8b7c05f |
| Files in original archive | 110 |
| Uncompressed bytes | 1,916,345 |
| Git metadata | Not included; archive comment and checksum manifests are provenance evidence, not a .git ancestry proof |

The source archive is the accepted P0-REVIEW-FIX-02 code snapshot. Do not substitute dd299d9, an earlier review package, or the later convenience bundle.

## 2. Important correction about the convenience migration bundle

The previously generated convenience bundle CTFworker-P0-REVIEW-FIX-02-migration-bundle.zip contains 119 files, but it is **not byte-for-byte the original source snapshot**:
- root README.md was replaced with the new repository navigation README;
- docs/ARCHITECTURE.md was replaced with the new target architecture document;
- the new governance documents were added.

This was intentional for a combined handoff bundle, but it must not be used as the exact source input. For source import, use the original 110-file archive. Preserve the new target README.md and docs/ARCHITECTURE.md already on the docs/project-baseline branch; rename the originals to README-P0-ORIGINAL.md and docs/reference/WORKBENCH_ARCHITECTURE.md.

## 3. Checksum manifest audit

### P0 solver manifest

docs/P0_SOLVER_SHA256SUMS contains 32 entries:
- 24 match raw bytes;
- 6 match after normalizing CRLF/LF to LF;
- 2 do not match: README.md and docs/P0_SOLVER_FILES.md.

All listed backend/app, backend/tests and solver_demo.py entries match either raw or LF-normalized content. Do not rewrite the historical manifest to conceal the two documentation mismatches.

### KAT manifest

docs/STREAM_CONTENT_V1_KAT_SHA256SUMS: all 4 entries match after LF normalization.

### Older MVP backend manifest

docs/MVP_BACKEND_SHA256SUMS is an earlier baseline manifest, not the P0-REVIEW-FIX-02 authority. In the supplied current snapshot, 17 entries match after LF normalization and 4 do not. Do not use this older manifest to reject the accepted P0 Manager Core snapshot; retain it as historical evidence.

## 4. Public-repository hygiene findings

Two historical evidence documents contain local Windows workspace paths:
- docs/a00/REGRESSION.md
- docs/MVP_WRITEUP_ACCEPTANCE.md

Before public import, replace only the absolute local path strings with neutral placeholders such as <LOCAL_WORKSPACE_PATH> and <TEMP_WORKSPACE_PATH>. Preserve commands, test results and surrounding evidence. Do not publish local usernames, machine paths, environment dumps, challenge data, real Flags, credentials or tokens.

Synthetic Flag strings in unit tests and solver_demo.py are deliberate fixtures and are labelled synthetic; they are not real challenge data. Keep them unchanged because they test the Flag-redaction contract.

## 5. Local validation performed against the extracted source

Environment:
- Linux container, Python 3.13.5;
- FastAPI 0.128.2, Uvicorn 0.48.0, httpx 0.28.1, Pydantic 2.13.5, pytest 9.0.2;
- Ruff unavailable;
- Node 22.16.0, npm 10.9.2;
- pinned project versions differ from this environment.

Results:
- Offline demo: **58/58 passed**.
- Backend tests as root: **533 passed, 2 skipped, 4 failed**. Three failures are read-only permission assertions because root can write despite Unix mode bits; one is a Windows-only junction test invoking cmd.exe, unavailable on Linux.
- Backend tests as an unprivileged user, excluding only the Windows-only junction test: **536 passed, 2 skipped, 1 deselected**. The three permission assertions pass when run without root privileges.
- Frontend: npm ci --offline could not proceed because vue-tsc 3.1.1 was not in the local npm cache. No frontend build result is claimed.
- Ruff: not run because Ruff is not installed in this environment.

These results show the synthetic demo and the non-Windows backend test set pass in this container, but they are **not** a green full migration acceptance. Re-run on the project's pinned Windows environment and/or configured CI.

## 6. CI portability issue to track separately

The supplied .github/workflows/ci.yml uses ubuntu-latest, while backend/tests/test_solver_review_regressions.py::test_08_junction_reparse_cross_task_access_rejected invokes Windows cmd /c mklink /J unconditionally. On Linux this raises FileNotFoundError instead of skipping.

Do not silently modify the P0 source during the import commit. Record this as a separate, narrowly scoped test portability fix after the exact source import, then require CI to pass before merge. The Windows junction test must remain covered on Windows; POSIX symlink tests remain separate.

## 7. Migration disposition

**Source identity: PASS WITH NOTES.** The original archive is correctly labelled and the P0 code/test manifest entries match; the archive does not carry Git metadata and two documentation manifest entries differ.

**Public hygiene: ACTION REQUIRED.** Redact the two local-path-bearing docs before commit.

**Runtime acceptance: INCOMPLETE.** Demo passes; backend tests pass under non-root Linux when the Windows-only test is excluded; Ruff and frontend build were not run; full Windows validation remains required.

## 8. Next task

Execute A0-2 using the original source archive:
1. import exact source files while preserving the new product north-star docs;
2. rename the original README and legacy Workbench architecture doc rather than overwriting the new repository docs;
3. redact local workspace paths in the two historical evidence docs;
4. run pinned Windows backend/Ruff/frontend checks and offline demo;
5. report CI portability issue separately;
6. open/update a migration PR and leave it unmerged for independent review.
