---
name: spec-write
description: Converts current conversation and repository context into a durable, reviewable feature spec under docs/specs. Use when the user asks to write, create, or update a Feature Spec, implementation spec, review contract, or Ralph-loop-ready task/spec document.
---

# Spec Write

Convert the current conversation and repository context into a single durable **Feature Spec** under `docs/specs`.

Do not implement the feature. Do not create a PRD. Do not publish to the issue tracker.

## Process

1. **Gather context without interviewing by default**
   - Use the current conversation as the primary source.
   - Explore relevant repository files when needed to understand current behavior, vocabulary, constraints, related specs, and validation commands.
   - If `CONTEXT-MAP.md` exists, read it and then the relevant context file. Otherwise read root `CONTEXT.md` when present.
   - Use canonical glossary terms from domain context, but do not edit glossary files.
   - Ask one concise clarification only if the feature name, scope, or target spec file cannot be inferred safely.

2. **Choose the target file**
   - Create `docs/specs` if it does not exist. Do not create unrelated docs directories.
   - Derive a kebab-case `<feature-slug>` from the feature name.
   - Use `docs/specs/YYYY-MM-DD-<feature-slug>.md`, where `YYYY-MM-DD` is the current date.
   - If today's matching file exists, update it.
   - If no same-day file exists but older `*-<feature-slug>.md` files exist, ask whether to update an older spec or create today's new spec.
   - If multiple older matches exist, ask which to update or whether to create today's new spec.

3. **Create or update the spec**
   - If creating, use the template below.
   - If updating, read the existing spec first and merge new information into the right sections.
   - Preserve existing requirements, scenarios, constraints, validation expectations, implementation context, and boundaries unless the current context clearly supersedes them.
   - Do not add implementation task ledgers or generic review checklists.
   - Summarize what was created or changed.

## Spec template

```md
# <Feature Name>

## Purpose

<One or two paragraphs explaining why this feature exists and what durable behavior the implementation must provide.>

## Requirements

### Requirement: <Behavior name>

<Subject> SHALL <externally observable behavior>.

#### Scenario: <Concrete scenario name>

- **WHEN** <trigger or state>
- **THEN** <observable result>
- **AND** <additional observable result, if needed>

## Implementation Constraints

- <Stable, review-relevant technical constraint. Omit this section if none are known.>

## Implementation Context

- <Implementation-relevant context from planning, grilling, existing code, rejected approaches, or known risks. Omit this section if none is needed.>

## Validation Expectations

- <Feature-specific validation guidance, such as required commands, testing boundaries, or acceptable manual validation. Omit this section if validation can be derived safely during implementation.>

## Out of Scope

- <Behavior the implementation must not introduce.>

## Source Context

- `<path-read-and-materially-used>`
```

Omit `## Source Context` if no repository files materially informed the spec. Omit `## Implementation Constraints` when no stable constraints are known. Omit `## Implementation Context` when there is no planning context worth preserving. Omit `## Validation Expectations` unless there is feature-specific validation guidance.

## Quality rules

- Requirement statements use normative `SHALL` language.
- Scenarios use `WHEN`, `THEN`, and optional `AND` bullets.
- Specify externally observable behavior; avoid implementation details unless they are stable review constraints or important implementation context from planning.
- Avoid vague acceptance criteria like “works well”, “is robust”, or “handles errors” without concrete scenarios.
- Do not include a detailed implementation task ledger. The implementation agent should derive its own plan from requirements, scenarios, constraints, and current code.
- Do not include a generic review checklist. Review agents should check the implementation against requirements, scenarios, constraints, out-of-scope boundaries, and validation evidence.
- Use `## Implementation Context` only for non-obvious planning context that future implementers need: resolved trade-offs, rejected approaches, migration risks, compatibility constraints, or other durable handoff notes.
- Use `## Validation Expectations` only for feature-specific guidance. Do not repeat generic testing policy in every spec.
- When validation guidance is needed, prefer the smallest safely discoverable commands from project files such as `package.json`, `Justfile`, `Makefile`, `pyproject.toml`, or `flake.nix`; do not invent commands.
- When tests are called for, describe the behavior to validate rather than internal mechanics. Avoid brittle assertions such as incidental CSS classes, HTML tag structure, snapshots, private methods, or internal collaborator call counts unless those details are the public contract.
- Make `## Out of Scope` explicit. If only a generic boundary is known, use: `Unspecified behavior changes outside this feature.`

## Guardrails

- Do not write application code.
- Do not publish or update issue tracker entries.
- Do not create PRDs.
- Do not create a separate task file or task ledger.
- Do not silently overwrite an existing spec; merge or ask when ambiguous.
