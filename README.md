# CTFworker — Local-first CTF Solver

CTFworker is the engineering home for a local-first, evidence-first CTF solving workbench.

> **Product promise:** one Manager coordinates one independent Worker per challenge; every factual Writeup claim traces back to recorded actions and evidence.

## Read this first

1. [Product blueprint](docs/PRODUCT_BLUEPRINT.md) — authoritative product framework, settled behavior and Product P0–P3 gates.
2. [Product charter](docs/PRODUCT_CHARTER.md) — users, workflow and non-negotiable principles.
3. [Architecture](docs/ARCHITECTURE.md) — current P0 code and target ACP/Host architecture.
4. [Roadmap](docs/ROADMAP.md) — Product, Engineering and Environment tracks.
5. [Current status](docs/STATUS.md) — current state and one next task.
6. [Agent workflow](docs/AGENT_WORKFLOW.md) — mandatory instructions for DeepSeek, Codex and reviewers.
7. [Decision log](docs/DECISIONS.md) — durable decisions and supersessions.
8. [Security policy](docs/SECURITY.md) — authorization, isolation, evidence and data handling.

## Product invariants

- One task = one independent ACP Worker Session = one task workspace.
- At most two active tasks; further tasks queue FIFO.
- CLI/headless first; GUI screenshots supported, autonomous clicking not required.
- G0 environment doctor and version-pinned `toolchain.lock` before formal solving.
- Manager coordinates; Worker solves; Worker explores before escalating.
- Five categories: Web, Misc, Crypto, Reverse and Pwn.
- SQLite + CAS are the sole evidence authority; Writeups derive from recorded evidence.
- Human approval for sensitive actions; no arbitrary targets or Flag-submission brute force.

## Current engineering baseline

P0-REVIEW-FIX-02 is the accepted Manager Core source candidate, labelled `a4d0762e1b5b7673a09ab5a2ac99656bc743925d`. Its original 110-file source archive is being migrated into this repository on the draft baseline PR. The import is not accepted until target-repository validation and independent review complete.

P0 Manager Core uses a deterministic Mock Runner for its current acceptance surface. It does not prove a real ACP process, live model provider, real binary execution, target connectivity, Docker isolation or platform Flag verification.

## Development rules

- Read the eight documents above before planning or editing code.
- Work on one numbered task at a time.
- Do not increase the two-task hard limit or replace the selected ACP Worker model without a new approved decision.
- Do not let Workers install software or modify the host during a solve.
- Keep real challenge data, Flags, tokens, raw logs, target URLs/IPs and generated Writeups out of this public repository.
- No merge, release, environment mutation or live-target run without explicit authorization.

## Offline synthetic demo

After the P0 source import:

```powershell
python scripts/solver_demo.py
```

The demo is synthetic and uses the Mock Runner. It does not launch a real Agent, execute challenge binaries or access a target.
