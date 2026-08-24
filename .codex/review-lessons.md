# Historical Review Lessons

这些是从人工 review 中提炼的可复用经验，而不是原始评论归档。仅在当前
PR 的变更会触及对应场景时应用；每条相关 lesson 都应在计划中给出验证证据。

## Contract and Compatibility

### RL-001 Accepted contracts are the source of truth

Do not simplify, reinterpret, or partially reimplement an accepted public contract.
Align schema, model, validator, examples, fixtures, exported APIs, and reason codes
to the latest accepted source before extending an adjacent feature.

When relevant: public schemas, cross-package APIs, contract migrations, or a branch
that may predate related contract work.

Required verification: compare the changed public surface against the accepted
contract and rebase/merge the relevant current base before declaring the PR ready.

### RL-002 Compatibility is an end-to-end release property

Version support matrices, dependency pins/locks, release tags, documentation, and
E2E assertions must describe the same artifact. Never move a published tag; publish
a new version when a public algorithm or compatibility surface changes.

When relevant: package upgrades, release candidates, cross-package dependencies, or
version-gated behavior.

Required verification: run the dependent package and cross-layer compatibility tests
against the exact resolved version; inspect lockfiles and version assertions.

### RL-003 Integrate stacked work in dependency order

Do not develop or review a PR against a stale base when its contract or dependency
PRs have already changed. Resolve the base first, then keep each stacked PR limited
to its own incremental responsibility.

When relevant: stacked PRs, merge conflicts, broad contract edits, or suspiciously
small diffs replacing larger accepted implementations.

Required verification: inspect the merge-relevant diff after rebase and run checks
against the resulting base, not an obsolete green CI run.

## Validation and Domain Invariants

### RL-004 Validate state-dependent fields as a matrix

Validate the allowed combination of status, reason code, retry metadata, candidates,
withdrawals, and other conditional fields. Checking each field's local format is not
enough. For example, a withdrawal may be valid only for permitted outcome states;
non-success, duplicate, or terminal states must not silently carry side effects.

When relevant: outcome/result validators, state machines, terminal transitions, or
fields whose validity depends on another field.

Required verification: add table-driven positive and negative cases for every
permitted and forbidden state/field combination, including duplicate and retry paths.

### RL-005 Enforce semantic invariants at every public input boundary

Use the complete validator or strict parser for mappings, raw JSON, and object entry
points. Reject malformed/oversized/non-finite/deep inputs with stable domain errors;
do not silently project an invalid object into a valid-looking one or leak raw parser
exceptions.

When relevant: deserialization, mapping/object constructors, projection APIs, or
alternative validation entry points.

Required verification: exercise equivalent valid and invalid inputs through each
public entry point, including limits, type errors, unknown fields, and parser errors.

### RL-006 Preserve domain distinctions and classification rules

Do not infer domain semantics from similarly named fields. Privacy classification,
retention, processing status, policy version, and reason code each have separate
contracts. Unknown forward-compatible versions may require quarantine rather than
rejection; invalid types and missing values remain invalid.

When relevant: classification/retention/privacy rules, policy versions, reason codes,
or compatibility behavior.

Required verification: test each semantic dimension independently and test the
specified treatment of unknown versus malformed values.

### RL-007 Derive aggregates and identities from their authoritative inputs

Collection members must be unique where the contract requires it; summaries, digests,
quality counters, and identifiers must be recomputed from the full authoritative
data. A stale summary, a forged cross-user ID, or a truncated input used for a
structural decision is a correctness defect.

When relevant: aggregate summaries, windows, IDs, digests, truncation, or
multi-tenant objects.

Required verification: cover duplicates, cross-user substitution, post-update
aggregate changes, and cases where a relevant item appears beyond a display limit.

### RL-008 Compare and canonicalize time as time

Parse timestamps into a normalized UTC representation before ordering, min/max,
identity generation, or scheduling. String ordering and differing fractional-second
formats must not change ordering or deterministic IDs.

When relevant: timestamps, cursors, retry deadlines, checkpoints, canonical bytes,
or replay identities.

Required verification: include equivalent UTC/offset forms, differing fractional
precision, naive timestamps, and reversed chronological inputs.

## State, Replay, and Privacy

### RL-009 Make replay deterministic and protect terminal history

Persist the exact revision/snapshot needed to reproduce an accepted result. Keep
high-water marks and terminal/tombstone state independent of the live namespace so
old or conflicting inputs cannot revive deleted state or become incorrectly stale.

When relevant: event replay, rebuilds, state revisioning, removals, supersession, or
idempotency receipts.

Required verification: replay after later revisions, test equal-revision conflicts,
out-of-order delivery, removal followed by old input, and byte/ID stability.

### RL-010 Privacy removal must erase all content-bearing replicas

A privacy remove applies to current state, replay inputs, ledgers, checkpoints,
compacted records, and database side files—not only the visible projection. Preserve
only the minimum non-content identity/tombstone data necessary for deterministic
behavior.

When relevant: deletion, retention expiry, privacy revocation, replay, or compaction.

Required verification: test every permitted remove reason, repeated/multi-item
removal, restart/replay, and absence of removed content from database, WAL/SHM, and
replay material.

### RL-011 Secure storage boundaries against aliasing and residual data

Validate resolved paths and database identity, reject untrusted symlinks/hard links,
use restrictive directory/file permissions, and enable secure deletion where the
store holds sensitive content. Do not rely only on path-string checks.

When relevant: SQLite/filesystem stores, state isolation, temporary paths, or privacy
data.

Required verification: cover symlink/alias attempts, wrong-store targets, required
permissions, secure-delete settings, and supported platform path behavior.

### RL-017 Bounded views must preserve decision provenance

Do not derive a privacy, retention, terminal, or other semantic effect solely from
a bounded projection window. If the retained view can omit the relevant record,
the trusted event/delta must carry the minimal validated, content-free reason
needed by the consumer; otherwise the consumer must not checkpoint an effect it
cannot prove. A generic fallback reason is incorrect when downstream behavior or
audit semantics distinguish the causes.

When relevant: projection windows, pagination, summaries, tombstone retention,
bounded publications, terminal withdrawals, privacy/retention removals, or any
consumer that checkpoints a result derived from partial data.

Required verification: trace the field through schema, model, producer, identity,
consumer, and golden fixtures. Test each semantic reason with the record retained
and omitted by the bounded view; assert the exact emitted reason and checkpoint
behavior in both cases.

## Transactions and Delivery

### RL-012 Commit a logical change atomically

Domain state, membership/revision changes, receipts, ledgers, outbox records, and
migration metadata must commit or roll back together. Run integrity checks before
commit; a failed post-condition must not leave a partially advanced schema or state.

When relevant: migrations, outbox workflows, multi-table updates, or failure
injection paths.

Required verification: inject failures after each material write and assert that all
affected state, receipts, outbox records, and migration versions remain consistent.

### RL-013 Scope claims, retries, and idempotency to the real owner

Workers must claim only supported event types/schema versions; receipts and uniqueness
keys must include the dimensions that define idempotency (such as user and builder
version). Retry only known transient failures. Lost leases must not overwrite another
worker's result or mask the original error.

When relevant: queues, outbox consumers, leases, retries, worker version upgrades, or
multi-user event processing.

Required verification: test unsupported events remain unclaimed, permanent failures
do not loop, transient failures retry, lease loss preserves the root failure, and
versioned reprocessing does not collide.

### RL-014 Defer late events instead of mutating closed history

If a late input would alter a closed, superseded, or otherwise terminal object, do
not force it through the live mutation path. Record a deterministic deferred outcome
and leave the later rebuild/revision/supersession workflow to the component that owns
it.

When relevant: late or out-of-order events, closed state, replay, or IDs derived from
earliest input.

Required verification: show that the terminal object remains unchanged, the source
input is retained, the receipt records deferral, and the outbox completes without a
silent retry or error object.

## Verification and Delivery Discipline

### RL-015 Test the claimed behavior, not a nearby happy path

An E2E or regression test is useful only when it creates the condition named in its
title and asserts the resulting artifact/side effect. Include split/merge, concurrent
writers, multi-user isolation, replay, and negative paths when those are part of the
contract.

When relevant: newly added E2E tests, rebuilds, concurrency, or bug-fix regressions.

Required verification: prove the setup produces the intended change, isolate prior
pending work, and assert the exact receipt, output, side effect, and no-op/replay
behavior.

### RL-016 CI must execute the real merge gate

Green partial tests do not validate a PR when required workflows are absent, stale,
conflicted, or omit format/lint/type/build/platform checks. Keep local commands and
CI gates aligned, and repair formatting and whitespace before review.

When relevant: CI/workflow edits, new packages, formatting failures, cross-platform
filesystem behavior, or a PR with incomplete checks.

Required verification: confirm the relevant workflow ran for the merge-relevant base
and execute the required targeted tests, full checks, lint, formatting, build, and
platform-specific smoke tests where applicable.
