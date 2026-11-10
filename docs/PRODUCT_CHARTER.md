# Product Charter

## 1. Product definition

**CTFworker is a local-first, evidence-first CTF solving workbench with a Manager/Worker execution model.** It helps an individual or small team manage authorized CTF challenges without mixing context, files, permissions, execution history or evidence.

It is not merely a prompt collection, a generic chat wrapper, a writeup generator, or an autonomous attack bot. Its value is the controlled workflow connecting a challenge to a bounded Worker and every claimed result back to recorded evidence.

## 2. User-facing workflow

1. **Create a challenge/task.** Record name, category, description, event metadata, optional screenshot and ZIP attachments, and explicitly authorized environment details.
2. **Intake and validate.** Preserve original attachment bytes, calculate hashes, validate archives, build a manifest, and provision a task-specific workspace.
3. **Schedule.** Admit no more than two active tasks by default. Additional tasks wait FIFO.
4. **Delegate.** Manager owns one Worker session per task through the Runner abstraction. A Worker receives only its task context and workspace contract.
5. **Analyze and solve.** Worker follows the relevant category playbook, forms hypotheses, performs authorized experiments, records observations and reports blockers.
6. **Authorize sensitive actions.** Network access, remote targets, exploit execution and other high-risk actions require explicit human authorization within a narrowly scoped policy. A Worker cannot approve itself.
7. **Record evidence.** Persist events and artifact references as work happens. Original inputs are immutable; derived artifacts are content-addressed and hash-verifiable.
8. **Review outcome.** Manager records solved/unsolved/failed/cancelled outcomes. Flag state distinguishes candidate, user-confirmed and externally verified; never imply platform verification when none exists.
9. **Generate Writeup.** Render from persisted events and artifacts, include references, and explicitly mark gaps or unverified claims.
10. **Export.** Produce a portable archive with manifest and SHA-256 checksums. Human edits remain distinct from generated drafts and are versioned.

## 3. Challenge categories

The product covers Web, Misc, Crypto, Reverse and Pwn. Category knowledge belongs in separate, lazily loaded playbooks/Skills. Manager routes work; it does not absorb specialist reasoning.

## 4. Roles

- **User / human operator:** owns authorization, scope, final Flag confirmation and acceptance.
- **Manager:** intake, queue, Worker lifecycle, approvals relay, state recovery, progress summary and final handoff. It should not perform heavy challenge analysis itself.
- **Worker:** one task, one context, one task workspace; analyzes and reports evidence and blockers.
- **Runner / ACP adapter:** execution lifecycle boundary. Manager depends on contracts, not a specific runtime.
- **Collector / evidence store:** authoritative persistence and artifact integrity; agents cannot rewrite history.
- **Reviewer:** independently verifies code, security invariants, tests and claims.

## 5. Non-negotiable principles

1. One challenge, one Worker, one workspace.
2. Two active tasks by default; excess tasks FIFO. Changes require a documented decision and resource evidence.
3. Workers request permissions; they never self-grant them. Approval and execution are distinct.
4. Writeup statements require persisted evidence or an explicit human/unverified label.
5. History is append-only. Corrections add records; they do not silently rewrite past actions.
6. Original attachments are immutable and their workspace copies are re-hashed.
7. Unknown permission, ambiguous target scope, invalid evidence, failed shutdown or integrity mismatch fails closed.
8. Challenge material and secrets remain local unless the user explicitly authorizes a specific outbound transfer.
9. Synthetic tests, Mock Runner, live provider calls, real target execution and platform verification are separate evidence classes.
10. Work proceeds in small, reviewable increments.

## 6. First-release non-goals

- Autonomous attack against arbitrary Internet hosts.
- Attacking competition platforms, brute-forcing Flag submissions, denial of service, or activity outside organizer authorization.
- Multi-user identity, team tenancy and cloud synchronization unless separately approved.
- A second full-stack UI duplicating the existing Workbench.
- Treating Skill prompts as a security boundary.
- Claiming OS/container isolation from a different working directory alone.
- Claiming real ACP execution from Mock Runner tests.
