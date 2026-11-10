# P0 Source Migration Guide

The documentation baseline is on branch `docs/project-baseline`. The accepted P0 source snapshot is supplied separately in the migration bundle; it has not yet been committed to this GitHub repository.

## Source provenance

- P0-REVIEW-FIX-02 source SHA: `a4d0762e1b5b7673a09ab5a2ac99656bc743925d`
- Historical reported tests: 532 passed, 7 skipped; 26 new regressions passed; demo 58/58; Ruff passed.
- Re-run tests after import. Historical numbers are not fresh target-repository results.

## Safe import procedure

1. Clone the target repository.
2. Fetch and branch from `origin/docs/project-baseline`.
3. Extract the migration bundle to a temporary directory.
4. Copy the bundle contents to the repository root, preserving `.git`.
5. Review all changed/staged paths. Exclude challenge data, Flags, tokens, raw logs, local paths and environment dumps.
6. Run `python scripts/solver_demo.py`, backend tests and Ruff; record actual output and skips.
7. Commit on the feature branch. Do not merge before independent review.

```powershell
git clone https://github.com/Sunset-Sunrise-Sun/CTFworker.git
cd CTFworker
git fetch origin docs/project-baseline
git switch -c migration/p0-review-fix-02 origin/docs/project-baseline
# Extract the migration bundle elsewhere, then copy its contents here.
git status --short
git diff --stat
python scripts/solver_demo.py
python -m pytest -q
```

No live ACP, live provider, real challenge or target network test is implied by the P0 baseline.
