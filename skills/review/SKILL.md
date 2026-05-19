---
name: review
description: Review the changes since a fixed point (commit, branch, tag, or merge-base) along two axes — Standards (does the code follow this repo's documented coding standards?) and Spec (does the code match the originating issue/PRD/spec and any relevant project specs?). Runs both reviews in parallel sub-agents and reports them side by side. Use when the user wants to review a branch, a PR, work-in-progress changes, or asks to "review since X".
---

# Review

Two-axis review of the diff between `HEAD` and a fixed point the user supplies:

- **Standards** — does the code conform to this repo's documented coding standards?
- **Spec** — does the code faithfully implement the originating issue / PRD / spec, while respecting relevant project specs?

Both axes run as **parallel sub-agents** so they don't pollute each other's context, then this skill aggregates their findings.

Repo setup should have been provided to you — run `/setup-skills` if `docs/agents/issue-tracker.md` or `docs/agents/specs.md` is missing.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point — a commit SHA, branch name, tag, `main`, `HEAD~5`, etc. Don't be opinionated; pass it through. If they didn't specify one, ask: "Review against what — a branch, a commit, or `main`?" Don't proceed until you have it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

### 2. Identify the spec sources

Look for the originating spec, in this order:

1. A path the user passed as an argument.
2. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`, etc.) — fetch via the workflow in `docs/agents/issue-tracker.md`.
3. A PRD/spec file matching the branch name, feature, or issue title. Use `docs/agents/specs.md` if it exists; otherwise search `docs/specs/`, `docs/`, `specs/`, and `.scratch/`.
4. If nothing is found, ask the user where the originating spec is. If they say there isn't one, the **Spec** sub-agent still checks related project specs when available, but reports "no originating spec available".

Then identify **related specs** that may constrain the change:

- Read `docs/agents/specs.md` when present to find the specs location and conventions.
- Search that location, plus `docs/specs/` as a fallback, for specs related to the branch name, touched areas, feature terms, or changed public behavior.
- Include the originating spec in this set if it is itself a repo spec.
- Do not treat unrelated specs as requirements; pass only plausibly relevant specs to the Spec sub-agent, and say how you chose them.

### 3. Identify the standards sources

Anything in the repo that documents how code should be written. Common locations:

- `CLAUDE.md`, `AGENTS.md`
- `CONTRIBUTING.md`
- `CONTEXT.md`, `CONTEXT-MAP.md`, per-context `CONTEXT.md` files
- `docs/adr/` (architectural decisions are standards)
- `.editorconfig`, `eslint.config.*`, `biome.json`, `prettier.config.*`, `tsconfig.json` (machine-enforced standards — note them but don't re-check what tooling already checks)
- Any `STYLE.md`, `STANDARDS.md`, `STYLEGUIDE.md`, or similar at the repo root or under `docs/`

Collect the list of files. The **Standards** sub-agent will read them.

### 4. Spawn both sub-agents in parallel

Send a single message with two `Agent` tool calls. Use the `general-purpose` subagent for both.

**Standards sub-agent prompt** — include:

- The full diff command and commit list.
- The list of standards-source files you found in step 3.
- The brief: "Read the standards docs. Then read the diff. Report — per file/hunk where relevant — every place the diff violates a documented standard. Cite the standard (file + the rule). Distinguish hard violations from judgement calls. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** — include:

- The diff command and commit list.
- The path or fetched contents of the originating spec, if found.
- The paths or contents of related specs, plus the reason each was considered relevant.
- The brief: "Read the originating spec and related specs. Then read the diff. Report: (a) requirements the originating spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong; (d) conflicts with relevant specs or spec review checklists. Quote the spec line for each finding. Under 400 words."

If no originating spec or related specs are available, skip the Spec sub-agent and note this in the final report.

### 5. Aggregate

Present the two reports under `## Standards` and `## Spec` headings, verbatim or lightly cleaned. Do **not** merge or rerank findings — the two axes are deliberately separate so the user can see them independently.

End with a one-line summary: total findings per axis, and the worst single issue (if any) flagged.

## Why two axes

A change can pass one axis and fail the other:

- Code that follows every standard but implements the wrong thing or violates a spec → **Standards pass, Spec fail.**
- Code that does exactly what the issue asked but breaks the project's conventions → **Spec pass, Standards fail.**

Reporting them separately stops one axis from masking the other.
