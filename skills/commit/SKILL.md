---
name: commit
description: Plans and creates Conventional Commits from the current git working directory, splitting changes into the smallest practical sequence of atomic commits. Use when the user asks to commit changes, create a conventional commit, commit the working directory, suggest commit messages, plan commits, or dry-run a commit split.
---

# Commit

Plan and create Conventional Commits from the current git repository changes. Optimize for reviewable, coherent commits while preserving user control.

## Core rules

- Inspect changes before committing.
- Propose a commit plan and ask for confirmation before staging or committing.
- Split changes into the smallest practical sequence of atomic commits.
- Order commits naturally: if one commit depends on another, it comes afterwards.
- Use Conventional Commit titles and a short prose body.
- Do not run validation by default.
- Do not amend, fixup, squash, rebase, or rewrite history.
- Do not warn or block based on branch name.
- Stop clearly if there is nothing to commit.

## Modes

### Commit mode

Use when the user asks to commit changes. Inspect changes, present the proposed plan, and ask for approval. After approval, execute all commits in plan order.

### Dry-run mode

Use when the user says "plan commits", "suggest commits", "dry run", "don't commit", or similar. Display the commit plan and stop without staging or committing.

## Inspect repository state

1. Confirm the current directory is inside a git repository.
2. Record the current branch with `git branch --show-current` for display only.
3. Inspect status with `git status --short`.
4. If there are no staged, unstaged, or relevant untracked changes, stop with:
   `No staged or unstaged changes found. Nothing to commit.`
5. Inspect staged changes with `git diff --staged` when present.
6. Inspect unstaged tracked changes with `git diff` when present.
7. Inspect untracked file names with `git ls-files --others --exclude-standard`.

## Source selection

Respect the index as intentional:

- If staged changes exist, treat staged changes as the commit candidate by default.
- Warn when unstaged changes also exist.
- Ask before including unstaged changes in a staged-change plan.
- If nothing is staged, plan from the whole working directory.

Untracked files:

- Include untracked files that clearly belong to the same change set, such as new tests, modules referenced by modified code, docs for the edited feature, or new skill files requested by the user.
- Ask before including untracked files that appear unrelated, generated, local-only, secret-like, or ambiguous.
- Flag suspicious files before inclusion: large files, binaries, build outputs, logs, archives, lockfiles, generated files, and secret-looking paths.

## Commit splitting

- Prefer the smallest practical number of atomic commits.
- An atomic commit is a coherent reviewable unit, not necessarily the theoretical minimum hunk-level split.
- Do not require patch-level splitting inside a file.
- If one file contains changes that could conceptually belong to multiple commits, commit the file as one unit rather than using fragile hunk staging.
- Group by purpose, behavior, module, and dependency order.
- Avoid mixing unrelated concerns when they can be staged safely as separate file/change groups.

## Conventional Commit messages

Use this format:

```txt
<type>(<optional-scope>): <imperative summary>

<short prose description of the intent of this commit.>
```

Guidelines:

- Infer the Conventional Commit type from the diff.
- Use common types such as `feat`, `fix`, `docs`, `test`, `refactor`, and `chore`.
- Avoid scope unless clearly supported by repository/module naming or a dominant changed area.
- Ask only if the type itself is ambiguous enough to risk a misleading commit.
- Keep the body intent-only; do not list files in the actual commit body.

## Fixed plan template

Always display the plan using this shape. Use `Notes: None` when there are no notes.

```md
## Proposed Commit Plan

Mode: <dry-run | will commit after approval>
Source: <staged changes only | whole working directory>
Branch: <current-branch>
Total commits: <n>
Validation: Not run by skill

### Commit 1: <type(scope): title>

Body:
<short intent-only commit body>

Purpose:
<1-2 sentence explanation>

Includes:
- <path or change group>
- <path or change group>

Depends on:
- None

Notes:
- <warnings, suspicious files, untracked ambiguity, or "None">

### Commit 2: <type(scope): title>
...
```

Do not show exact commands by default. Show command previews only when the user asks for verbose output or asks to see the commands.

## Snapshot and approval

- The user's approval applies only to the inspected working tree/index snapshot.
- Before creating commits, re-check that the relevant status/diff still matches the approved snapshot.
- If the snapshot changed, abort and rebuild the plan.
- Ask for confirmation once for the whole plan before executing.

## Execution

After approval:

1. Stage exactly the paths/change groups for each approved commit.
2. When operating on the whole working directory, rebuild the index as needed to match each planned commit.
3. When operating in staged-only mode, do not silently pull in unstaged changes.
4. Create each commit in dependency order.
5. If any `git add` or `git commit` command fails:
   - Stop immediately.
   - Report the failed command.
   - Summarize the error.
   - Show the current `git status`.
   - Do not continue to later commits.

## Validation

Do not run tests, linters, formatters, builds, or other validation commands by default. If the user explicitly asks for validation, run only the requested checks and report the results separately from the commit message.
