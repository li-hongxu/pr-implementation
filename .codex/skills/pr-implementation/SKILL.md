---
name: pr-implementation
description: Implement a scoped PR from requirements and acceptance criteria through verification and an independent $code-review pass. Use for end-to-end PR implementation, not for review-only, PR publishing, merging, or long-term review-memory work.
---

# PR Implementation

Own the local implementation lifecycle for the PR described in the user's current
message. Treat the supplied requirements, acceptance criteria, constraints,
non-goals, and other notes as the PR context; do not require the user to first put
them in a file. Do not push, open, modify, merge, or otherwise manage a remote PR.

Follow applicable repository instructions, especially the nearest-scope `AGENTS.md`.
Do not modify `AGENTS.md` unless the user separately asks. If present, read
`.codex/review-lessons.md` and apply only lessons relevant to this change. It is
reference material, not a task to update or a substitute for repository discovery.

## 1. Frame the PR

Before editing, turn the PR context into a working checklist:

- functional requirements and acceptance criteria;
- explicit constraints, compatibility expectations, and reliably stated non-goals;
- likely architecture and correctness risks.

Proceed when the supplied context is sufficient. Ask only for a material missing
decision, such as a requirement whose plausible interpretations would produce
incompatible behavior. Keep the checklist current as facts emerge.

### Semantic-boundary checklist

Before implementation, identify every boundary where one layer claims to trust
another: committed publication, authorization, identity, version/revision,
privacy classification, or persisted effect. Record the *proof* accepted at
that boundary, not merely the fields that name it. A non-empty ID, a frozen
dataclass, canonical JSON, or a private constructor alone is not proof that an
external transaction committed.

For each boundary, plan at least one adversarial test that changes exactly one
claimed fact, such as a forged publication ID, mismatched digest, rebinding of
two otherwise valid documents, or cross-user object ID. A boundary backed by
another service needs either a validated witness produced by its adapter or an
explicit integration test against that adapter. Do not represent metadata as a
trusted witness without a verification path.

When a decision depends on a bounded projection, window, summary, or other
lossy view, trace every semantic discriminator it needs back to an authoritative
trusted input. A display limit must not silently downgrade a semantic effect—for
example, a privacy withdrawal must not become an ordinary removal merely because
its tombstone is outside the retained window. Either carry the bounded,
content-free decision metadata in the trusted event/delta and validate it, or
quarantine/defer without advancing the checkpoint when the effect cannot be
proved. Test each semantic reason both retained and omitted by the bounded view.

## 2. Discover the Repository

Read the applicable instructions, relevant implementation and tests, existing
abstractions/patterns, and likely indirectly affected modules before changing code.
Use narrowly targeted Git history only when it could explain a local design or a
past regression; never require the user to provide history or read it wholesale.

Identify the intended PR base for later review. Prefer an explicitly supplied target
branch; otherwise use an unambiguous tracking/default branch. Confirm it resolves
and that `git diff <base>...HEAD` is meaningful. If no reliable base can be found,
ask for it before the code-review stage and do not claim final completion.

## 3. Plan and Implement

Create a concise requirement-to-delivery map before coding:

| Requirement / criterion | Implementation area | Verification |
| --- | --- | --- |
| ... | ... | ... |

Include anticipated files/tests, the highest-risk behavior, special acceptance
checks, and applicable review lessons. For every relevant lesson, record its ID,
the invariant it guards, and concrete verification evidence in this map. Add a
regression test when the changed code can exercise the lesson's scenario; otherwise
state why the scenario is not reachable. Keep it in the working conversation unless
a repository convention requires a planning artifact.

When a contract has numbered scenarios or a state machine, add a compact
requirement-to-test matrix. Each scenario must name its fixture, expected
effect/no-effect, and assertion. A broad test with many assertions is not
evidence that every scenario was covered unless the matrix identifies the
specific assertion for each one.

For a cross-layer contract field, map its complete path before coding:
authoritative schema and validator, domain model, producer, signed/identified
payload, consumer, examples, and golden fixtures. A consumer fallback is not a
fix when the producer cannot supply the semantic value. If one dimension can be
truncated or omitted, make the test matrix cross that condition with every
reason/status whose downstream effect differs.

For comparisons and aggregation, include cross-dimensional fixtures instead of
only happy-path values. Vary independent revisions separately, and use at least
two heterogeneous sources for every aggregate field. Set-like aggregate fields
must be checked for their documented operation, their cardinality limit, and
preservation of every permitted source value.

Implement only the scoped change. Reuse established abstractions and public
contracts; preserve backward compatibility unless the PR explicitly permits a
breaking change. Do not conceal design violations just to satisfy tests, broaden the
scope with unrelated refactors, or leave debug/temporary workarounds. Revise the
plan if discovery invalidates it.

## 4. Verify Requirements

Run the most relevant targeted tests first. Then run the repository-appropriate
additional checks: affected integration/e2e tests, full tests, lint, typecheck,
formatting, and build checks where available and proportionate to the change.

Independently assess every requirement and acceptance criterion as `VERIFIED`,
`PARTIALLY VERIFIED`, or `NOT VERIFIED`, with the evidence used. Passing tests alone
do not prove the PR correct. A critical `NOT VERIFIED`, failed required check, or
unresolved correctness risk blocks completion.

Treat a fixture as suspect when two independent domain quantities happen to
share the same value. Add a discriminating fixture before relying on the test;
for example, make an episode revision differ from its enclosing State revision.
For multi-layer behavior, verify both the pure component and the real adapter
boundary when the latter supplies trust or persistence.

## 5. Require an Independent `$code-review`

Do not create a second review framework or replace this step with a self-review.
Invoke the existing `$code-review` skill after verification, using the resolved PR
base as its fixed point. Give it the current PR context in the hand-off: requirements,
acceptance criteria, explicit constraints/non-goals, the full merge-relevant diff
command (`git diff <base>...HEAD`), relevant surrounding implementation/tests,
applicable repository instructions, and the relevant excerpts from
`.codex/review-lessons.md`.

Use an invocation equivalent to:

```text
$code-review review since <base>

PR requirements and acceptance criteria:
<current PR context>

Review context: inspect git diff <base>...HEAD and the relevant code/tests.
Also consider these applicable repository instructions and review lessons:
<relevant excerpts>
```

`$code-review` owns its standards/spec review methodology. This skill only supplies
context, interprets its findings, and controls remediation. If `$code-review` is not
available or cannot run because the base/diff is invalid, report that as an unmet
final-gate condition rather than silently continuing.

Do not bias the reviewer by saying that a suspected issue is already fixed or
by narrowing it to a proposed implementation. Ask the reviewer to construct
adversarial inputs at every trust boundary and to check the requirement-to-test
matrix against the actual fixtures.

## 6. Remediate and Recheck

For actionable review findings, determine the cause, make a scoped fix, rerun
targeted tests plus any affected broader checks, and reassess affected acceptance
criteria. If the fix is substantive, run `$code-review` again against the same PR
base. A finding is not resolved until that independent recheck has inspected
the remediation; self-assessment and a passing regression alone do not close it.

Use at most three complete review passes in one implementation run. Stop earlier
when the review has no unresolved blocker or major correctness issue. If the final
allowed pass finds a blocker, do not claim completion after an unreviewed fix:
report the PR as incomplete and require a later independently reviewed continuation.

## 7. Final Gate and Report

Before completion, inspect `git diff` and `git status`. Confirm the diff is scoped,
includes expected tests, and has no accidental files, debug code, temporary
workarounds, or unrelated edits.

Declare the PR complete only when all core requirements and acceptance criteria are
`VERIFIED`; relevant tests and required repository checks pass; `$code-review` ran;
and no blocker, major correctness issue, known architecture violation, or obvious
regression remains. Otherwise give a blocked/incomplete result and name what remains.

Before declaring completion, explicitly confirm that every trust boundary has
proof-backed verification, every numbered acceptance scenario has a matrix row
or an explicit out-of-scope owner, and every substantive remediation received
an independent review pass.

Use this concise completion report:

## Implemented

<core delivered behavior>

## Requirement Verification

<each key requirement/criterion and its VERIFIED, PARTIALLY VERIFIED, or NOT VERIFIED status>

## Validation

<commands/checks actually run and their results>

## Code Review

<$code-review result, important fixes, and any remaining blocker/major finding>

## Remaining Risks

`No known blocking issues.`

Replace that last line with explicit unresolved issues whenever any exist.
