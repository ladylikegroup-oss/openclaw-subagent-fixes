# Validation and evidence boundaries

This document describes what was actually checked, rather than treating build success, a child's finished flag, or delivery status as complete task success.

## Source integrity — verified 2026-11-09

- GitHub's official annotated `v2026.10.5` tag resolves to `1b58cb9a1b4f4353e1add63c8506db4d890962e8`.
- The repaired source checkout and the deployed installation's build metadata both identify `a02d409a4c751ecd83239ed71aae948d1c868bcd` and version `2026.10.5`.
- The exported patch changes 23 files across six original repair/test commits, listed in `VERSION.json`.
- `git apply --cached --check` passed against a separate index initialized from the exact upstream base.
- Applying the patch to that index reproduced the **complete repaired Git tree**, not only selected files.
- Reverse-check and reverse-apply reproduced the exact original base tree. Neither the source worktree nor its normal index was changed.
- No prebuilt installation archive or internal incident log is included in the publication.

## Publication-day focused regression

**2026-11-09: 122 tests passed in 9 files across 4 routed shards; exit code 0.** Shard counts were 14 + 67 + 37 + 4, with no skipped or failed cases in this selection. Wall time was **68.41 seconds**, on macOS arm64 with Node **26.3.0** and pnpm **12.9.0**, against exact repaired source `a02d409a`. This includes the real requester-wake admission and legacy no-repeat regressions.

One initial test launcher referenced a nonexistent pnpm JavaScript entry and exited before any tests ran. It was corrected to the installed executable for the same pinned pnpm version; no product code or assertion was changed. That launcher error is not counted as a test result.

The command below is the reproducible nine-file selection, using upstream's test router:

```sh
pnpm install --frozen-lockfile
pnpm test --maxWorkers=1 \
  src/agents/agent-command.peer-followup.test.ts \
  src/agents/bash-tools.exec-authorization.integration.test.ts \
  src/agents/subagents/announce/subagent-announce.requester-settle-admission.test.ts \
  src/agents/subagents/announce/subagent-announce.requester-settle-results.test.ts \
  src/agents/subagents/announce/subagent-announce.requester-settle-wake.test.ts \
  src/agents/subagents/registry/subagent-requester-settle-identity.test.ts \
  src/infra/exec-approvals-schema-admission.test.ts \
  src/process/supervisor/service-child-relay-host.startup.test.ts \
  src/process/supervisor/service-child-relay-host.test.ts
```

Use a fresh synthetic test environment, not a production state directory. These tests require no live model credentials. Do not disable transport, CA, preload, integrity, or authorization protections to get a passing test.

## Reproduction contracts

### Oversized parent wake identity

`subagent-announce.requester-settle-admission.test.ts` drives the requester-settle wake through the native announce/delivery and in-process agent-admission path. Model-provider I/O is controlled by fixtures; the identity builder, validator, delivery, and admission boundary are not bypassed.

Original red/green evidence included:

- Four UUID children with execution-identity audit on: passed in the fixture.
- Five UUID children with audit off: passed in the fixture.
- Five UUID children with audit on: failed on the original bounded contract, then passed after repair.
- An already accepted legacy input, reloaded through SQLite and a replaced Gateway runtime, was not run a second time.
- Fresh pause notices do not prematurely version one member of a multi-child completion cohort.
- Unsupported/mixed versions, cancellation, retry, and generation boundaries retain their rejection/isolation behavior.

The final migration scenario preserves a single model invocation but does not manufacture successful delivery when durable delivery evidence is missing.

### Peer follow-up admission

`agent-command.peer-followup.test.ts` reaches `agentCommandFromGatewayIngress` and holds a reply owner before its durable fence exists. Ordinary peer work waits; internal/restart exceptions retain their existing behavior. Historical incident attribution remains separate from this reproduced defect.

### Refused launch cleanup

The service-child relay tests and exec-authorization integration test exercise pre-launch refusal and native cleanup settlement. The command remains refused, the original refusal is preserved, and the prepared POSIX child is settled without manufacturing successful execution. Post-launch failures and Windows job-anchor handling are not relaxed.

## Historical repair verification — 2026-11-03 / 2026-11-04

These are separate scoped results, **not additive totals and not fresh publication-day executions**:

- Integrity diagnostics: 58 focused tests passed.
- Peer admission candidate: 74 focused/sibling tests passed; scoped lint/format/type checks passed.
- Refused-launch cleanup candidate: 66 focused/sibling tests passed; scoped lint/format/type checks passed.
- Versioned completion-identity candidate: 251 distinct cases passed across its scoped runs, including migration, replay, pause, and cancel coverage; scoped lint/format/type checks passed.
- The nine changed-test selection passed 122 tests before the final test-only fixture follow-ups.
- Final test-fixture follow-ups passed 150 cases with one existing Windows-only skip in their own selection. This is not the same selection as the nine-file command above.
- Final repaired source `a02d409a` completed build and package validation. The build took about 113 seconds and packaging/validation about 196 seconds.
- Operator smoke after the original installation confirmed Gateway/Discord operation, a native command's normal exit, and a native child completion event returning to its parent. This publication does not rerun that live smoke or certify every tool surface.

## Full-suite and platform limits

The original broad run was **not green**: it was interrupted with 740 planned shards not started, and subsequent database-worker/infra runs had retained failures. Focused fixture repairs do not retroactively turn those exits into success. Broad checks also reported pre-existing max-lines/ratchet/tooling issues. A UI startup gzip-size check reported 374,195 bytes versus a 373,454-byte limit despite overall build success; no independent base-build comparison established whether this patch caused that difference.

This publication verifies a source patch and focused regressions, not a new official binary release. Linux and Windows hardware, every `sessions_spawn` path, later OpenClaw releases, and all historical incident root causes are outside the verified scope.
