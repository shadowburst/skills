---
name: pr
description: Creates a GitHub pull request for the current branch onto the base branch, using branch diff, commits, and linked specs for context. Use when the user asks to create, draft, open, or prepare a PR / pull request for the current branch.
---

# PR

Create a review-ready GitHub pull request for the current branch. Package the branch for review; do not perform a full code review. Use `thermos` when the user asks for review before PR creation.

## Process

### 1. Inspect repository and branch state

- Confirm the current directory is a git repository.
- Determine the current branch with `git branch --show-current`.
- Refuse to create a PR from the base branch itself.
- Check for uncommitted changes. If present, briefly note that they will be ignored and continue; PR context must come only from committed branch state.
- Verify the remote is GitHub. If the remote is not GitHub, do not attempt creation; produce a complete PR title/body draft and explain that automatic creation currently requires a GitHub remote.

### 2. Determine the base branch

Prefer sources in this order:

1. Explicit base branch from the user.
2. GitHub default branch via `gh repo view --json defaultBranchRef`.
3. Local `origin/HEAD`.
4. `main`.
5. `master`.

Report the chosen base. Ask only if it cannot be determined safely.

### 3. Check publishing and existing PRs

- Check for an existing PR with `gh pr view --json url,title,baseRefName`.
  - If one exists, report it and ask whether the user wants to update/open it instead of creating a duplicate.
- Check whether the branch has an upstream remote.
  - If not pushed, ask before running `git push -u origin <branch>`.
- Do not silently push branches or create duplicate PRs.

### 4. Gather PR context

Capture committed branch context only:

- Diff against the base using `git diff <base>...HEAD`.
- Commit list using `git log <base>..HEAD --oneline`.
- Changed files using `git diff --name-only <base>...HEAD`.

Find a linked spec in this order:

1. Use an explicit spec path if the user provides one.
2. Search `docs/specs/` for filenames whose slug overlaps the current branch name after removing common prefixes such as `feature/`, `feat/`, `fix/`, `chore/`, and `ralph/`.
3. If exactly one strong match exists, use it.
4. If multiple plausible matches exist, ask the user to choose.
5. If no strong match exists, inspect recent commits and changed paths for additional terms, then ask whether the PR is linked to a spec.
6. If the user says no, proceed with `No linked spec`.

Do not treat weak similarity as authoritative; wrong spec context is worse than no spec context.

### 5. Infer the Conventional Commit title

Infer the PR title from the branch diff and commit list, preferring the dominant user-facing change over mechanical details.

Priority:

1. If the branch has one meaningful conventional commit, reuse its title.
2. If multiple commits share one obvious type/scope, synthesize one title.
3. Otherwise infer from changed behavior:
   - `feat:` for new capability
   - `fix:` for bug fix
   - `docs:` for documentation-only
   - `test:` for tests-only
   - `refactor:` for behavior-preserving restructure
   - `chore:` for tooling/maintenance
4. Include a scope only when obvious from repo/module names.
5. Ask before creating the PR if the title remains ambiguous.

### 6. Build the PR body

Use this reviewer-oriented structure:

```md
## Purpose

<Why this branch exists / what behavior or maintenance goal it delivers.>

## Changes

- <Review-relevant change>
- <Review-relevant change>

## Requirements Covered

- <Spec requirement or inferred requirement>

## Review Notes

- <Known tradeoffs, risky areas, migration notes, or "None">

## Linked Spec

- `<path>` or `No linked spec`
```

Add optional sections only when relevant: `## Out of Scope`, `## Screenshots`, `## Follow-ups`.

### 7. Confirm and create

Show the resolved base branch, linked spec, title, and full body draft. Ask for explicit confirmation before running `gh pr create` unless the user explicitly asked to create without confirmation.

Create with:

```sh
gh pr create --base <base> --head <branch> --title '<title>' --body '<body>'
```

If the user requested a draft PR, include `--draft`. Do not infer reviewers, labels, assignees, milestones, projects, or issue-closing keywords unless explicitly provided.
