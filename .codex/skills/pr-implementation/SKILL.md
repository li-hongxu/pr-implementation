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
checks, and applicable review lessons. Keep it in the working conversation unless a
repository convention requires a planning artifact.

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

## 6. Remediate and Recheck

For actionable review findings, determine the cause, make a scoped fix, rerun
targeted tests plus any affected broader checks, and reassess affected acceptance
criteria. If the fix is substantive, run `$code-review` again against the same PR
base.

Allow at most three complete review passes in total. Stop earlier when the review has
no unresolved blocker or major correctness issue. If the limit is reached with a
blocker or major issue remaining, stop and report it clearly; never present the PR as
complete.

## 7. Final Gate and Report

Before completion, inspect `git diff` and `git status`. Confirm the diff is scoped,
includes expected tests, and has no accidental files, debug code, temporary
workarounds, or unrelated edits.

Declare the PR complete only when all core requirements and acceptance criteria are
`VERIFIED`; relevant tests and required repository checks pass; `$code-review` ran;
and no blocker, major correctness issue, known architecture violation, or obvious
regression remains. Otherwise give a blocked/incomplete result and name what remains.

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
