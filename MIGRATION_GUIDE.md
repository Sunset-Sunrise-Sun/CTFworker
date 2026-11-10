# P0 Source Migration Guide

The product/governance baseline is on branch docs/project-baseline (Draft PR #1). The accepted P0 source code must be imported from the **original source archive**, not the later convenience bundle.

## Exact source artifact

- Outer package: `CTF-P0-review-fix-02-package.zip`
- Exact nested archive: `CTF-P0-review-fix-02-package/04-source-snapshot/ctf-manager-p0-fix-02-a4d0762-source.zip`
- The earlier shorthand filename `ctfworker_p0_a4d0762_source.zip` was inaccurate; do not search for that basename.
- ZIP comment/source label: a4d0762e1b5b7673a09ab5a2ac99656bc743925d
- SHA-256: ef523c473ad1c047687191e10e93e7deed40db809e1670813b67a7d8f8b7c05f
- 110 files; 1,916,345 uncompressed bytes
- The inner ZIP has been extracted from the outer review package and independently verified against the SHA-256 above.
- The archive contains no .git metadata; its comment is a provenance label, not a Git ancestry proof.

Do not use CTFworker-P0-REVIEW-FIX-02-migration-bundle.zip as the exact source: that convenience bundle replaced root README.md and docs/ARCHITECTURE.md with new governance versions and added files.

## Protected target files

Do not overwrite the target branch's README.md, MIGRATION_GUIDE.md, docs/PRODUCT_BLUEPRINT.md, docs/PRODUCT_CHARTER.md, docs/ARCHITECTURE.md, docs/ROADMAP.md, docs/STATUS.md, docs/AGENT_WORKFLOW.md, docs/DECISIONS.md, docs/SECURITY.md or docs/MIGRATION_AUDIT_20261110.md.

Rename the original source files:
- source README.md -> target README-P0-ORIGINAL.md
- source docs/ARCHITECTURE.md -> target docs/reference/WORKBENCH_ARCHITECTURE.md

Redact local absolute paths in docs/a00/REGRESSION.md and docs/MVP_WRITEUP_ACCEPTANCE.md before public commit. Preserve synthetic Flag fixtures in tests.

## Execution task

Follow docs/tasks/A0-2_P0_SOURCE_IMPORT.md exactly. It defines import mapping, non-goals, test commands, CI portability caveat, stop conditions and required report.

## Acceptance

- Preserve P0 source behavior and frozen interfaces.
- Run pinned Windows backend/Ruff/frontend checks and offline demo.
- Report actual outputs and skips; do not claim unrun checks.
- Keep migration PR draft/open/unmerged for independent review.
- Do not start real ACP integration or toolchain installation in this task.
