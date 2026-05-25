---
name: spec-plan
description: Runs an interactive planning session that turns an idea into either a durable Feature Spec or a chat-only Session Plan. Use when the user asks to plan a feature, grill an idea, decide whether work needs a spec, or prepare bounded implementation work.
---

# Spec Plan

Turn the user's idea into either a durable lean **Feature Spec** or a bounded chat-only **Session Plan**, but only after an interactive grilling session.

Use the `grill-with-docs` behavior for grilling. Use `spec-write` for Feature Spec creation and update mechanics.

Do not implement while grilling or while writing/updating a Feature Spec. A Session Plan may be implemented only after presenting it, receiving explicit user confirmation, and passing the clean-tree gate.

## Operating mode

- Ask one question at a time and wait for the user's answer.
- If a question can be answered by inspecting the repository, inspect the repository instead of asking.
- Read relevant `CONTEXT.md`, `CONTEXT-MAP.md`, and ADRs when needed.
- Update `CONTEXT.md` inline when durable domain terms are resolved.
- Create ADRs only when the `grill-with-docs` criteria are met: hard to reverse, surprising without context, and a real trade-off.
- Do not force ADRs or glossary entries for transient implementation details.

## Post-grilling decision

After the grilling session reaches stable understanding, classify the work and explain the recommendation.

Recommend a **Feature Spec** when the work changes durable behavior, requirements, constraints, out-of-scope boundaries, domain language, architectural decisions, cross-session decisions, durable behavior contracts, or non-obvious constraints.

Recommend a **Session Plan** when the work does not introduce or change a durable behavior contract, domain term, cross-session decision, or non-obvious constraint.

Then ask the user to choose:

1. Write/update a Feature Spec.
2. Create a Session Plan, then choose whether to implement it.

If the user chooses Feature Spec, immediately write or update the Feature Spec through `spec-write`; do not ask an additional redundant “write/update now?” confirmation.

If the user chooses Session Plan, immediately present the `## Session Plan` block and ask whether to implement it now.

If you recommend a Feature Spec and the user chooses immediate Session Plan implementation, warn what durable context may be lost by skipping the Feature Spec, then ask for explicit confirmation before implementing.

## Feature Spec branch

Use `spec-write` target-file behavior for path selection, creation, and update mechanics. After writing or updating the Feature Spec, self-check it:

- title exists
- `## Purpose` exists
- `## Requirements` exists
- at least one `### Requirement: ...`
- at least one `#### Scenario: ...`
- `## Out of Scope` exists
- no generic validation boilerplate
- no implementation task ledger
- no generic review checklist

Identify the planning files changed by this planning session:

- always include the Feature Spec that was created or updated
- include any `CONTEXT.md`, context-specific glossary, or ADR files changed during this planning session
- do not include unrelated dirty files that were not changed by this planning session

Ask whether to commit those planning files. Invite the user to review or amend them first, for example: “Review or amend the files now if desired. Commit these planning files?”

If the user wants to inspect or amend first, pause for their next instruction and do not commit until they explicitly confirm.

If the user declines the commit, end with `Created/updated: <spec-path>` and explain that implementation is not ready until the planning changes are committed, stashed, or cleaned. Do not claim readiness for implementation.

If the user confirms the commit:

- stage only the planning files identified above with `git add -- <planning-files...>`
- if the Feature Spec is new, use a Conventional Commit message equivalent to `docs(specs): add <feature-slug> spec`
- if the Feature Spec already existed, use a Conventional Commit message equivalent to `docs(specs): update <feature-slug> spec`
- do not refuse solely because a touched planning file had pre-existing uncommitted edits; commit the current contents of that touched file if the user confirmed
- if `git commit` fails, report the failure and do not claim implementation readiness

After a successful planning commit, run `git status --porcelain`. If unrelated dirty files remain, warn that implementation may refuse to start until those changes are committed, stashed, or cleaned.

End only after a successful planning commit with:

- `Created/updated: <spec-path>`
- `Ready for implementation with spec-implement: <spec-path>`

## Session Plan branch

Present a short chat-only block in this shape:

```md
## Session Plan

No Feature Spec is warranted because: <reason>

Scope:
- <what will change>

Acceptance:
- <observable expected result or done condition>

Validation:
- <commands/checks to run, or why none applies>
```

Rules:

- A Session Plan is ephemeral and must not be written to `docs/specs` or another planning-file directory.
- Keep it short; do not include Feature Spec requirements/scenarios sections.
- If durable domain language or a durable decision is discovered while preparing a Session Plan, update `CONTEXT.md` or offer an ADR according to the normal `grill-with-docs` rules. The Session Plan itself remains chat-only.

After presenting the Session Plan, ask for explicit confirmation to implement it now.

If this planning session changed `CONTEXT.md`, a context-specific glossary, or an ADR, identify those planning files and ask the user to commit, stash, or otherwise clean them before implementation.

If the user confirms immediate implementation, use `spec-implement` with the accepted Session Plan as the implementation contract. `spec-implement` owns the clean-tree gate, implementation execution, validation, review/fix pass, and final implementation report. Do not create a planning-file copy of the Session Plan.
