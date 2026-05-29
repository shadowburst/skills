---
name: implement
description: Implement a spec, accepted plan, current instruction, or review-change list using gathered context, TDD when useful, sub agents where available, validation, and a thermos-style review/fix loop. Never commits.
---

# Implement

Implement a bounded change. Accept any of these as the implementation contract:

- an explicit Feature Spec path
- an accepted current-session plan
- a clear current instruction with scope and acceptance expectations
- a user review-change list, such as “implement these review changes”

Do not commit, create a PR, or write process logs into specs.

## Before editing

1. Identify the implementation contract.
2. If a Feature Spec path is provided, read it and treat it as the authority.
3. If no Feature Spec path is provided, proceed only when the current instruction or accepted plan is clear enough to implement. Otherwise ask for clarification.
4. Inspect the repository state for awareness, but do not block because of uncommitted changes.
5. Note the starting revision or diff base for the final report.

Dirty working trees are context, not a blocker. Avoid unrelated dirty files where practical. If the change must touch an already-dirty file, proceed and mention it in the final report.

## Spec policy

When implementing from a Feature Spec:

- The spec is the behavior contract.
- Do not silently change requirements, scope, or out-of-scope boundaries.
- Amend the spec only when the user explicitly asks, or for tiny non-behavioral corrections needed for handoff accuracy.
- If the requested implementation contradicts the spec, stop and ask whether to amend the spec or implement different behavior.

## Execution

Use sub agents where available.

Preferred flow:

1. Context gatherer: read the contract, relevant docs, relevant code, and validation sources.
2. Planner: derive the smallest safe plan and decide whether TDD applies.
3. Worker: implement the plan.
4. Validation: run relevant deterministic checks.
5. Review/fix loop: run a thermos-style review, apply required fixes, and repeat up to 3 total review iterations.
6. Final validation.
7. Short final report.

Use TDD when the contract requires tests, changes observable behavior with a discoverable test harness, or regression risk justifies coverage. Prefer public/user-observable assertions over implementation-detail assertions.

## Review/fix loop

- Run a thermos-style review after implementation.
- Review for correctness, regressions, security/devex risks, maintainability, simplicity, and contract compliance.
- Apply only required fixes.
- Repeat up to 3 total review iterations.
- Stop early when no required fixes remain.
- Do not chase optional polish unless explicitly requested.

## Validation

- Run relevant discoverable validation commands.
- If validation fails, fix the issue or report failure.
- If validation is unavailable or inapplicable, explain what was attempted and what manual validation remains.
- If the contract requires tests and they cannot be run, stop with failure.

## Completion rules

- Do not commit.
- Do not create a PR.
- Do not invent task ledgers or update task checkboxes unless the user explicitly asks.
- Leave changes in the working tree for user review and commit.

## Final report

End with:

```md
## Summary
<what changed>

## Validation
- <commands/results or why not run>

## Review
- <iterations run>
- <required fixes applied or none>

## Changed Files
- `<path>` — <why>
```
