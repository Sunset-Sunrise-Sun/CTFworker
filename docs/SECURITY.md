# Security, Authorization and Data Handling

## 1. Authorized-use boundary

Use only CTF challenges, practice targets and infrastructure explicitly authorized by the user and within organizer/owner rules.

Never attack or scan arbitrary public hosts; attack competition platforms, scoreboards, registration or submission infrastructure; brute-force Flag submissions; disrupt other participants; pivot outside declared scope; or continue after authorization expires, becomes ambiguous or is revoked.

A target address in a prompt, attachment or challenge statement is not proof of authorization. Record scope and authorization separately.

## 2. Permission model

- Worker can request permission; it cannot grant permission to itself.
- Approval and execution are separate transitions.
- Grants are narrow, explicit, time-bounded and task-bound; include target/domain/IP, port, protocol, action and expiry as applicable.
- Task switch never carries approval from another task.
- Unknown, expired, mismatched or downgraded permission state fails closed.
- Any live target/network execution requires explicit user authorization and a separate acceptance plan.

## 3. Isolation model

A unique cwd is organizational isolation, not OS security isolation. Harness tool policy, Skill text, working directory and separate session do not provide kernel-enforced filesystem, process, network or resource isolation by themselves.

Before executing untrusted binaries or exploit code, require an OS/container boundary with task-specific mounts, read-only originals, non-root execution, minimal capabilities, no privileged container or host Docker socket mount, explicit egress policy, CPU/memory/process/disk/time limits, bounded output, process-tree termination and verified cleanup, plus escape/symlink/junction/mount/network tests.

If a required boundary is unavailable, the affected execution mode remains disabled. Do not relabel it safe.

## 4. Data classes

Never commit challenge attachments/extracted files, Flag values, credentials, tokens, cookies, private keys, raw session logs containing challenge material, non-public target addresses, real Writeups, screenshots/evidence from real challenges, local paths, usernames, hostnames, machine identifiers or environment dumps.

Allowed public content: source code, generic docs, tests, synthetic fixtures, redacted logs and non-sensitive checksums.

## 5. Evidence integrity

- SQLite is the authoritative metadata/event store in the current Workbench integration.
- CAS stores immutable bytes by SHA-256.
- Agent event payloads and candidate Flag plaintext must not be written directly to SQLite.
- Append corrections as new events; do not rewrite history.
- Writeup claims resolve to evidence or are labelled human-provided/unverified.
- Missing evidence makes export incomplete; do not silently fill gaps.
- Flag states never imply external platform verification unless a verifier performed it.

## 6. Stop conditions

Stop and report on unrecognized target/permission, path traversal or link-like path, hash mismatch, invalid/ambiguous ZIP, failed Worker stop confirmation, failed artifact/event transaction, unrecorded timeout/resource exhaustion, or environment policy block.

Do not bypass security controls by renaming files, changing paths, switching tools, disabling controls or using alternate sources.
