# OpenClaw 2026.10.5 — subagent handoff fixes

[繁體中文說明](README.zh-TW.md) · [Validation](docs/VALIDATION.md) · [Version manifest](VERSION.json)

A small, MIT-licensed **community patch set** for three reproduced lifecycle defects around subagent completion, parent wake-up, peer follow-up, and refused process launch. Includes the source changes, regression tests, diagnostics, and test-fixture corrections from the deployed repair.

**This is not an official OpenClaw release or an upstream-merged fix.** Sharing the same version does not mean every installation encounters these defects; the triggering conditions matter.

## Exact version

- Affected and verified base: **OpenClaw `2026.10.5`**, official tag [`v2026.10.5`](https://github.com/openclaw/openclaw/tree/v2026.10.5).
- Exact upstream base commit: `1b58cb9a1b4f4353e1add63c8506db4d890962e8`.
- Patch-package version: **`2026.10.5-fix.1`**. This versions this repository's patch, not the upstream npm package.
- Original repaired source commit: `a02d409a4c751ecd83239ed71aae948d1c868bcd`.
- Original deployed build ID: `2026.10.5-a02d409a4c75-2026-11-03T21-37-55.869Z`.
- Other OpenClaw versions, later upstream fixes, Linux, and Windows: **not verified by this publication**.

The original repaired commit is a provenance identifier, not a commit hosted in this small repository. Applying the patch to the exact upstream base reproduces its complete Git source tree. `VERSION.json` records both tree IDs and the SHA-256 of each changed file.

## What it fixes

### 1. Completed children cannot wake their parent: oversized execution identity

Observed error:

```text
execution identity admission token violates its bounded contract
```

The requester-settle producer concatenates child run IDs into a parent wake identity. With execution-identity auditing enabled, a sufficiently long identity exceeds the existing **256-character** contract. A regression fixture reproduces the failure with five UUID child IDs; four pass in that fixture. Five is not a universal threshold because other identity components also contribute to its length.

The patch bounds newly admitted completion identities using SHA-256 of the **whole** identity, and persists `identityVersion: 1` before dispatch. It preserves exact identities for previously attempted, unversioned cohorts so an upgrade cannot silently replay already accepted work. Unsupported or mixed identity versions are rejected. A pause notice cannot assign a version to an entire completion cohort.

### 2. Peer follow-up races with an active reply before its execution fence exists

A peer follow-up could claim command-recovery ownership while an existing reply already owned the session but had not yet written its durable execution fence. The patch waits for that reply owner while preserving command FIFO custody and the existing internal/restart exceptions.

This is a reproduced admission race. It is **not** evidence that every historical `SESSION_WORK_START_CHANGED` report has this cause.

### 3. A refused command launch is misreported as lost cleanup identity

An admission refusal before a command launch could discard the prepared POSIX relay's cleanup custody and obscure the original refusal with:

```text
service child cleanup identity lost
```

For this pre-launch POSIX case, the patch retains the prepared anchor until cancellation and native process extinction settle. It reports the original refusal. The Windows job-anchor path and post-launch identity-loss handling remain unchanged.

### Diagnostic and fixture support

Integrity-read refusals now distinguish revision mismatch, missing integrity context, and unavailable proof. **The integrity checks still reject invalid reads.** Test-only follow-ups make recovery release, Git process cleanup, npm metadata isolation, DNS, and APFS fixtures observe their actual contracts.

## Get and apply the source patch

Use a new source checkout in a durable working directory. Do not apply this to an installed `node_modules` directory or a running Gateway.

```sh
mkdir -p ~/src
cd ~/src
git clone https://github.com/ladylikegroup-oss/openclaw-subagent-fixes.git
git clone --depth 1 --branch v2026.10.5 --single-branch \
  https://github.com/openclaw/openclaw.git openclaw-2026.10.5
cd openclaw-2026.10.5
test "$(git rev-parse HEAD)" = "1b58cb9a1b4f4353e1add63c8506db4d890962e8"
git switch -c fix/subagent-handoff-2026.10.5
git apply --check ../openclaw-subagent-fixes/openclaw-2026.10.5-subagent-fixes.patch
git apply ../openclaw-subagent-fixes/openclaw-2026.10.5-subagent-fixes.patch
```

Run each step only if the previous step succeeds. Check the downloaded patch against [SHA256SUMS](SHA256SUMS). The patch includes **23 files**: production changes, tests, and test routing. Do not mix it with independently cherry-picked copies of the same fixes.

Use the Node range and pinned pnpm version in upstream `package.json` (pnpm **12.9.0** for this base), then follow the [test commands and evidence](docs/VALIDATION.md). Build from source using the upstream build workflow after your tests pass. This repository does not ship a prebuilt npm package or auto-updater.

## Deployment and rollback boundary

This publication changes no running installation. Test the patched source in an isolated environment before choosing to deploy. Use OpenClaw's supported operator update workflow, with a coordinated backup of the matching binary and state and an actual restore check.

Reversing this patch in a clean, unused source checkout is possible with `git apply --reverse`. That is **not** a safe runtime rollback procedure after the patched binary has written versioned lifecycle state; do not roll back only the package while retaining incompatible state.

## Known limits

- Already attempted, unversioned long cohorts keep their old identity. They are not automatically renamed, retried, or revived.
- Markerless long pause identities retain the previous audit-on length rejection.
- This does not add administrator permissions, change RLS, disable integrity validation, or raise token/identity limits.
- The repair does not establish a single common root cause for all past subagent or tool failures.
- Focused regression checks and a successful historical build are **not a full green CI run**. The original broad suite was incomplete and had retained failures; see the validation document.
- The original operator smoke verified native child completion and return-to-parent. It was not a comprehensive `sessions_spawn` or all-platform acceptance test.

## Contents and provenance

- `openclaw-2026.10.5-subagent-fixes.patch`: exact aggregate diff from the upstream base to the repaired source, without private commit messages or local paths.
- `VERSION.json`: pinned base, original source commits, source-tree identities, patch digest, and changed-file digests.
- `docs/VALIDATION.md`: reproduction boundaries, verified results, and remaining limits.
- `SHA256SUMS`: artifact checksums.

Only source changes and synthetic test fixtures are included. No deployment configuration, credentials, databases, customer data, conversation transcripts, or internal incident logs are published. When reporting a problem, include version/build ID, a sanitized error, and a minimal synthetic reproduction; do not upload your live state or secrets.

## License

[MIT](LICENSE). Upstream OpenClaw copyright is retained. Patch and documentation publication is maintained by [ladylikegroup-oss](https://github.com/ladylikegroup-oss).
