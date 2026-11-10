# Product Charter

## 1. Product definition

**CTFworker is a local-first, evidence-first CTF solving workbench with a Manager/Worker execution model.** It helps an individual or small team manage authorized challenges without mixing task context, files, permissions, execution history or evidence.

It is not a generic chat wrapper, standalone second Agent platform, Writeup fabricator or autonomous attack bot. Its value is the controlled workflow connecting a challenge to one bounded Worker and every claimed result to recorded evidence.

## 2. User-facing workflow

1. Submit a challenge, category, description, attachments and declared scope.
2. Validate/preserve inputs, hash them and create a task workspace.
3. Run the first-start G0 environment doctor before formal solving.
4. Admit at most two active tasks; queue further tasks FIFO.
5. Create one independent ACP Worker Session per task, with that task's work directory as workspace root.
6. Use CLI/headless analysis first and the matching category Skill.
7. Let the Worker explore reasonable safe approaches before escalating a genuine blocker.
8. Require explicit human approval for sensitive actions.
9. Record actual actions/results and immutable evidence as work happens.
10. Review Flag state and evidence-derived Writeup; export only after checking gaps and redactions.

## 3. Categories and solving posture

The product covers Web, Misc, Crypto, Reverse and Pwn. Each has a separate lazily loaded playbook:
- Misc: metadata/container/encoding/stego/traffic analysis; encrypted archives get format/password-hint analysis and bounded local attempts first.
- Crypto: mathematical reasoning first, supported by reproducible tool calculations.
- Reverse: static analysis first; dynamic execution only in a verified isolated environment.
- Pwn: local testing first; remote exploitation only after a one-time task/target/time-window approval.
- Web: source/configuration/local reproduction first; remote access only to explicitly authorized challenge targets.

## 4. Roles

- **Human operator:** owns scope, authorization, final Flag confirmation and acceptance.
- **Manager:** intake, routing, two-slot FIFO, Worker lifecycle, approval relay, recovery, progress and final report. It does not perform heavy solving.
- **Worker:** one task, one ACP session, one workspace; performs category analysis and reports evidence/blockers.
- **Runner / ACP adapter:** lifecycle boundary; Manager depends on its contract, not runtime-specific implementation.
- **Collector / evidence backend:** authoritative SQLite/CAS persistence; Agents cannot rewrite history.
- **Reviewer:** independently verifies code, security invariants, tests and claims.

## 5. Non-negotiable principles

1. One challenge = one Worker = one workspace.
2. Two active tasks maximum; further tasks queue FIFO.
3. CLI/headless first; GUI screenshot capture is supported but autonomous clicking is not required.
4. G0 prepares/freezes the baseline toolchain before formal solving; Workers never install host tools.
5. Workers request permissions; they never self-grant them.
6. Approval is task/target/action/time scoped and checked by the enforcement layer.
7. Evidence is append-only; corrections add records rather than rewriting history.
8. Writeup statements require evidence or an explicit human/unverified label.
9. Unknown permission, ambiguous target, invalid evidence, failed shutdown or integrity mismatch fails closed.
10. Real challenge material and secrets remain local unless the user explicitly authorizes a specific transfer.
11. Mock tests, synthetic tasks, live ACP, real binary execution, target interaction and platform Flag verification are distinct evidence classes.
12. Work proceeds in small, reviewable increments.

## 6. First-release non-goals

- Arbitrary Internet targets, competition-platform attacks, broad scanning or Flag-submission brute force.
- Multi-user identity, team tenancy or cloud sync unless separately approved.
- A second full-stack UI or second Agent loop.
- Treating Skills, cwd or ACP workspace as an OS sandbox.
- Autonomous GUI clicking as a required solve path.
- Claiming real ACP execution from Mock Runner tests.
