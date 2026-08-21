# PR Implementation for Codex

`pr-implementation` is a repository-local Codex skill that supervises the full
local delivery loop for a pull request: it turns requirements into a plan,
implements the scoped change, verifies acceptance criteria, requests an
independent review, fixes findings, and reports the final gate result.

It is deliberately small. It does not push code, create remote pull requests,
merge branches, or keep a database of review history.

## Why this exists

An implementation task is more than writing code that makes a test pass. A PR can
still miss an acceptance criterion, violate an established boundary, or leave an
unreviewed regression. This skill keeps requirements, verification, and independent
review connected from the first edit through completion.

## How it differs from `$code-review`

| `$pr-implementation` | `$code-review` |
| --- | --- |
| Orchestrates local PR delivery from requirements to final gate. | Independently evaluates an existing diff. |
| Plans, implements, runs checks, maps evidence to acceptance criteria, and fixes findings. | Separates standards and spec findings. |
| Must invoke `$code-review`; it does not duplicate review logic. | Is the review capability reused by this skill. |
| Does not publish, push, or merge a PR. | Does not implement the change. |

## Install in a repository

Copy the `.codex/` directory into the repository that should use the workflow:

```text
.codex/
├── review-lessons.md
└── skills/
    └── pr-implementation/
        └── SKILL.md
```

The skill assumes `$code-review` is available in the Codex environment.

## Use

Start a Codex conversation from the target repository and provide the current PR
context inline. A separate requirements file is optional.

```text
$pr-implementation

PR requirements:
- Add customerId filtering to order queries.
- Preserve the current response contract.

Acceptance criteria:
- A valid customerId returns only that customer's orders.
- Invalid values return the existing 400 response shape.
- Omitting customerId preserves current behavior.
- Tests cover the new behavior.

Constraints:
- Do not add dependencies.
- Do not refactor unrelated modules.

Base branch: main
```

The explicit base branch lets the independent review compare the merge-relevant
diff reliably. When omitted, the skill uses an unambiguous tracking/default branch
when possible and otherwise asks before the review gate.

## Review lessons

`.codex/review-lessons.md` stores short, generalized lessons distilled from past
human reviews. It is intentionally not a raw review archive or an automatic memory
system. During a relevant PR, the skill reads and applies these lessons while
planning and implementing; v1 does not update the file automatically.

## Delivery gates

The skill reports completion only after core requirements and acceptance criteria
are verified, applicable checks pass, `$code-review` has run, and no unresolved
blocker or major correctness issue remains. Review remediation is capped at three
complete review passes so unresolved risks are surfaced instead of hidden in an
endless loop.

## Scope

This is a local development workflow. It intentionally excludes remote PR actions,
automatic merging, GitHub-comment ingestion, reviewer subagents of its own, and
long-term memory infrastructure.
