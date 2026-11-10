# CTFworker — Local-first CTF Solver

CTFworker is the engineering home for a local-first, evidence-first CTF solving workbench. It coordinates challenge intake, per-challenge workspaces, bounded Worker execution, human authorization, immutable evidence, and Writeup/export.

> **Core product promise:** a Writeup is a traceable rendering of recorded actions and evidence—not a story reconstructed from a final Flag or an agent's recollection.

## Start here (mandatory reading order)

1. [Product blueprint](docs/PRODUCT_BLUEPRINT.md) — the agreed target architecture, end-to-end workflow and product P0–P3 gates. This is the anti-drift north star.
2. [Product charter](docs/PRODUCT_CHARTER.md) — product goal, users, scope and non-negotiable rules.
3. [Architecture](docs/ARCHITECTURE.md) — current P0 code, target Harness integration and component ownership.
4. [Roadmap](docs/ROADMAP.md) — product phases, engineering milestones and environment dependencies.
5. [Current status](docs/STATUS.md) — what is done, blocked, and the one next task.
6. [Agent workflow](docs/AGENT_WORKFLOW.md) — required instructions for DeepSeek, Codex and reviewers.
7. [Decision log](docs/DECISIONS.md) — durable decisions; do not silently reverse them.
8. [Security and data policy](docs/SECURITY.md) — authorized scope, isolation, secrets and evidence rules.

## Current implementation baseline

- **P0 Manager Core baseline:** source snapshot a4d0762e1b5b7673a09ab5a2ac99656bc743925d (P0-REVIEW-FIX-02).
- P0 provides challenge intake, per-task workspace, a two-slot FIFO scheduler, persisted task/Worker state, ACP v1 adapter contracts with a deterministic Mock Runner, SQLite event persistence, CAS artifact storage, and incremental Writeup generation.
- P0 does **not** prove a real ACP process, live model provider, real challenge execution, target connectivity, or competition-platform Flag verification.
- P0 was accepted with follow-up notes. See [status](docs/STATUS.md).

## Repository layout

- backend/app/solver/: Manager, scheduler, Runner/ACP contracts, intake, workspace, store and Writeup.
- backend/app/: existing Workbench integration and persistence primitives.
- frontend/: existing Workbench UI; no separate full-stack product is planned.
- docs/: product, architecture, roadmap, status, decisions, security and acceptance records.
- scripts/, tests/: reproducible checks and synthetic fixtures.

## Development rules

- Read the eight documents above before planning or editing code.
- Work on one numbered task at a time; do not expand scope without approval.
- Default concurrency is **two active challenge Workers**; excess tasks wait FIFO.
- Never execute an untrusted attachment or contact a target during mock-only acceptance.
- Keep real challenge data, Flags, tokens, raw logs, target URLs/IPs and generated Writeups out of this public repository.
- No push, PR merge, release, environment mutation or live-model run without explicit authorization.
- A green test suite is evidence for the tested scope only; it is not proof of production readiness.

## Offline demo

```powershell
python scripts/solver_demo.py
```

This demo is synthetic and uses the Mock Runner. It does not launch a real agent, execute challenge binaries, or access a target.
