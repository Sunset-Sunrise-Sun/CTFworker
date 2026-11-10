# Security, Authorization and Data Handling

## 1. Authorized-use boundary

Use only CTF challenges, practice targets and infrastructure explicitly authorized by the user and within organizer/owner rules.

Never attack or scan arbitrary public hosts; attack competition platforms, scoreboards, registration or submission infrastructure; brute-force Flag submissions; disrupt other participants; pivot outside declared scope; or continue after authorization expires, becomes ambiguous or is revoked.

A target address in a prompt, attachment or challenge statement is not proof of permission. Record scope and authorization separately.

## 2. Permission model

- Worker can request permission; it cannot grant permission to itself.
- Approval and execution are separate transitions.
- Grants are narrow, explicit, time-bounded and task-bound; include target/domain/IP, port, protocol, action and expiry as applicable.
- Pwn remote grants use window semantics: one approval permits a continuous bounded sequence for the same task and target within the approved time window. Completion/expiry invalidates it; no cross-task or cross-target reuse.
- Each protected operation rechecks the grant. A grant recorded in a prompt is not enforcement.
- Unknown, expired, mismatched or downgraded permission state fails closed.
- Generic web research and challenge-range traffic are separate channels. Target hosts are blocked from the auxiliary research channel; range traffic uses structured `ctf-range-*` operations with per-call scope checks.

## 3. Tool boundary

- Prefer Harness-native file/text/CLI/headless tools for ordinary work.
- One CTF Tools MCP server exposes structured heavy-tool APIs only; no arbitrary shell endpoint.
- MCP processes are outside the Harness shell sandbox/approval mechanism. Every MCP method must enforce task/Worker identity, authorization, target scope, schema validation, timeout, output limits and isolation itself.
- Skills are playbook instructions, not a security boundary.
- Cordis is a thin Host bridge; it must not become a second scheduler, task database or Agent loop.

## 4. Isolation model

A task-specific directory or ACP session workspace is an organizational/write-boundary control, not a complete OS security sandbox. Harness policy, Skill text, working directory and separate session do not provide kernel-enforced filesystem, process, network or resource isolation by themselves.

Before executing untrusted binaries or exploit code, require an OS/container boundary with task-specific mounts, read-only originals, non-root execution, minimal capabilities, no privileged container or host Docker socket mount, explicit default-deny egress, CPU/memory/process/disk/time limits, bounded output, process-tree termination and verified cleanup, plus escape/symlink/junction/mount/network tests.

If a required boundary is unavailable, the affected execution mode remains disabled. Do not relabel it safe.

## 5. G0 environment readiness

Before formal solving, run the approved environment doctor, verify baseline tools and versions, run self-tests and freeze them in `toolchain.lock`. Missing baseline tools are installed only through the approved environment workflow. Workers do not install software or alter the Windows host during a solve. Warm-up discoveries update the reviewed inventory and require a new doctor/lock record.

## 6. Data classes and evidence

Never commit challenge attachments/extracted files, Flag values, credentials, tokens, cookies, private keys, raw session logs containing challenge material, non-public target addresses, real Writeups, real screenshots/evidence, local paths, usernames, hostnames, machine identifiers or environment dumps.

Allowed public content: source code, generic docs, tests, synthetic fixtures, redacted logs and non-sensitive checksums.

SQLite is the only authoritative task/event store. CAS stores immutable bytes by SHA-256. Agent payload bytes and candidate Flag plaintext must not be written directly to SQLite. Append corrections as new events. Writeup claims resolve to evidence or are labelled human-provided/unverified. Missing evidence makes export incomplete. Flag states never imply external platform verification unless a real verifier performed it. Export excludes/redacts `flag_capture` and sensitive `agent_event_payload` artifacts by default.

## 7. Stop conditions

Stop and report on unrecognized target/permission, path traversal or link-like path, hash mismatch, invalid/ambiguous ZIP, failed Worker stop confirmation, failed artifact/event transaction, unrecorded timeout/resource exhaustion, unexpected missing toolchain dependency, or environment policy block.

Do not bypass controls by renaming files, changing paths, switching tools, disabling controls or using alternate channels.
