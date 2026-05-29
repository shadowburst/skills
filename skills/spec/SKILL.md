---
name: spec
description: Write or update a durable feature spec from available context, run a focused spec-quality review loop, then automatically commit only the planning files it changed.
---

# Spec

Turn the available context into a durable **Feature Spec** under `docs/specs/`. Do not grill, implement, create a PR, or publish to an issue tracker.

Use the current conversation, explicit user-provided context, relevant repository docs, and relevant code. Ask at most one concise clarification only when the feature name, scope, or target file cannot be inferred safely.

## Workflow

1. Gather only the context needed to write a correct handoff spec.
2. Choose the target file.
3. Create or update the spec.
4. Run a spec-quality review loop with at most 2 review iterations.
5. Apply required spec fixes from the review loop.
6. Automatically commit only the planning files changed by this run.

## Target file

- Create `docs/specs/` if needed.
- Derive a kebab-case `<feature-slug>` from the feature name.
- Use `docs/specs/YYYY-MM-DD-<feature-slug>.md`, using the current date.
- If today's matching file exists, update it.
- If only older matching specs exist, ask whether to update an older spec or create today's new spec.
- If multiple older matches exist, ask which to update or whether to create today's new spec.

## Spec shape

```md
# <Feature Name>

## Purpose

<Why this feature exists and what durable behavior it must provide.>

## Requirements

### Requirement: <Behavior name>

<Subject> SHALL <externally observable behavior>.

#### Scenario: <Concrete scenario name>

- **WHEN** <trigger or state>
- **THEN** <observable result>
- **AND** <additional observable result, if needed>

## Implementation Context

<Optional. Non-obvious context a fresh implementation session needs.>

## Validation Expectations

<Optional. Feature-specific validation guidance.>

## Out of Scope

<Behavior this feature must not introduce.>

## Source Context

- `<path-read-and-materially-used>`
```

Omit optional sections when they add no useful handoff context.

## Quality rules

- The spec must contain enough context for a fresh session to implement the feature.
- Requirements use normative `SHALL` language.
- Requirements and scenarios describe externally observable behavior.
- Scenarios use concrete `WHEN`, `THEN`, and optional `AND` bullets.
- `## Out of Scope` is explicit. If only a generic boundary is known, use: `Unspecified behavior changes outside this feature.`
- Include implementation context only for durable handoff information: constraints, rejected approaches, compatibility risks, trade-offs, or codebase context.
- Include validation expectations only when they are feature-specific or not safely discoverable.
- Do not include implementation task ledgers, generic checklists, PRDs, issue-tracker updates, or execution logs.

## Spec review loop

Run up to 2 spec-quality review iterations. Each iteration checks:

- Is there enough context for a fresh session to implement?
- Are requirements externally observable and unambiguous?
- Are scenarios concrete and meaningful?
- Are edge cases, constraints, and out-of-scope boundaries clear enough?
- Is implementation context useful without becoming a task plan?
- Is validation guidance sufficient without over-prescribing internals?

Apply required fixes. Stop early when no required fixes remain. Do not chase optional polish.

## Commit policy

After the review loop, commit immediately.

- Stage only planning files changed by this run: the spec, plus any `CONTEXT.md` or ADR files changed during the same run.
- Do not block because unrelated files are dirty.
- For a new spec, commit with: `docs(specs): add <feature-slug> spec`
- For an updated spec, commit with: `docs(specs): update <feature-slug> spec`
- If the commit fails, report the failure and stop.

## Final response

Report:

- spec path
- review iterations run
- commit hash or commit failure
