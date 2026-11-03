---
name: commit
description: Analyze current Git changes, group them into logically independent commits, propose Conventional Commit messages, and execute commits one group at a time after explicit user approval. Use when the user asks to commit changes, organize commits, split changes into separate commits, or create commit messages. Never stage or commit anything before the user approves the commit plan.
---

# Commit Command

Analyze, organize, and commit the current Git changes. Follow the workflow and safety rules below strictly.

## Core Principles

1. Group changes by intent and behavior, not merely by file or directory.
2. Each commit must represent one clear, cohesive, and independently understandable change.
3. Every commit message must accurately describe the actual changes being committed.
4. Never commit all changes together simply because they are currently in the working tree.
5. **Never run `git add`, `git commit`, or any command that changes the Git index before the user explicitly approves the proposed commit plan.**
6. Never commit `CLAUDE.md`, regardless of its status.
7. **Never generate a `Co-Authored-By` or `Co-authored-by` trailer in any commit message.**
8. Never discard, overwrite, or revert the user's existing changes.
9. Preserve unrelated working-tree changes and existing staged changes.
10. Prefer correctness, clear change boundaries, and data safety over execution speed.

## Workflow

### 1. Inspect the Current Git State

Start by inspecting the repository:

```bash
git status --short
git diff --stat
git diff
git diff --cached --stat
git diff --cached
```

Inspect individual files or additional diffs when necessary.

Identify and distinguish:

- Unstaged modifications
- Already staged changes
- Untracked files
- Added, deleted, and renamed files
- Changes belonging to unrelated work
- `CLAUDE.md` and other files that must not be committed

Important considerations:

- `git diff` does not include staged changes. Always inspect `git diff --cached` as well.
- Untracked files are not included in ordinary diffs. Inspect their contents when relevant.
- Do not assume existing staged changes belong to this task.
- Do not assume every modified file should be committed.
- Do not infer a change's purpose from its filename alone.
- If the ownership or intent of a change is unclear, ask the user before proceeding.
- Do not expose sensitive file contents unnecessarily when reporting the analysis.

### 1.5. Check Branch and Remote State

Before proposing a plan, check the current branch and its relationship with the remote:

```bash
git branch --show-current
git fetch --prune
git status -sb
git rev-list --left-right --count @{upstream}...HEAD
```

`git fetch` only updates remote-tracking refs and does not touch the working tree or index, so it counts as a read-only inspection command and may run before approval.

Identify and report:

- The current branch name, or a detached HEAD
- Whether the branch has an upstream, and which one
- How many commits the branch is ahead of and behind its upstream
- Whether the branch has diverged from its upstream
- Whether a merge, rebase, cherry-pick, or revert is in progress
- Whether the current branch is a shared development branch (see "Shared Branches" below)

How to handle the results:

- **Up to date or ahead only**: Proceed normally.
- **Behind or diverged**: Warn the user in the commit plan that committing now will create diverged history, and that a later push will require a pull, rebase, or merge. Ask how they want to proceed.
- **No upstream**: Note it in the plan. Do not set an upstream.
- **Shared development branch**: State it prominently at the top of the commit plan. Approving the plan counts as confirmation, so do not ask a separate question. Treat any behind or diverged state on these branches as a high-priority warning, because other team members push to them.
- **Detached HEAD or an operation in progress**: Flag it clearly and ask for confirmation before continuing.

#### Shared Branches

A shared development branch is one that other team members also push to. Determine which branches are shared as follows:

1. If CLAUDE.md (at any level) defines shared branches or branch rules, follow that definition.
2. Otherwise, treat `main`, `master`, `develop`, `dev`, and `release/*` as shared branches.

Run `git remote -v` and report the remote in the plan alongside the branch name.

- **Fetch fails** (offline, authentication, or other errors): Report the error, state that the remote state could not be verified, and ask whether to continue with local information only.

Never run `git pull`, `git merge`, `git rebase`, `git push`, `git branch --set-upstream-to`, or `git push -u` as part of this check. Synchronizing with the remote requires explicit user approval.

### 2. Group Changes by Intent

Organize changes into logically cohesive commit groups based on their purpose, behavior, scope, and nature.

Follow these grouping rules:

- Separate changes with different purposes.
- Generally separate features, bug fixes, refactoring, formatting, tests, documentation, and maintenance changes.
- Split different types of changes within the same page or component when they serve different purposes.
- Changes across multiple files or pages may belong in one commit if they serve the same cohesive purpose.
- If a single file contains unrelated changes, consider partial staging.
- Keep changes together when separating them would create an inconsistent, incomplete, or invalid intermediate state.
- Identify dependencies between groups and propose a safe commit order.
- Do not split changes artificially just to increase the number of commits.

For each group, determine:

1. What problem does it solve, or what purpose does it serve?
2. Why do these changes belong together?
3. Why should they be separated from other groups?
4. Can the group be committed independently?
5. Does it depend on another group?
6. Does it require partial staging?

### 3. Determine the Commit Type

Choose the type based on the primary purpose of the actual changes.

| Type       | Usage                                                                         |
| ---------- | ----------------------------------------------------------------------------- |
| `feat`     | Introduces a genuinely new feature or capability                              |
| `fix`      | Corrects a bug, incorrect behavior, unexpected result, or compatibility issue |
| `docs`     | Changes documentation without changing application behavior                   |
| `style`    | Makes formatting-only changes that do not affect program behavior             |
| `refactor` | Restructures code without introducing features or fixing bugs                 |
| `perf`     | Primarily improves performance                                                |
| `test`     | Adds or improves tests                                                        |
| `chore`    | Handles dependencies, tooling, build configuration, or maintenance            |
| `revert`   | Reverts a previous commit                                                     |

#### Type Selection Rules

**`feat`**

Use `feat` only when the change introduces functionality or a capability that did not previously exist.

Do not use `feat` merely because an existing feature, page, component, or code path has been modified or improved.

Examples that do not automatically qualify as `feat`:

- Optimizing an existing feature
- Updating an existing page or component
- Improving code structure
- Fixing an existing behavior
- Upgrading dependencies without introducing application functionality

**`fix`**

Prefer `fix` when the primary purpose is to correct incorrect logic, unexpected behavior, display problems, compatibility issues, or other bugs.

**`refactor`**

Use `refactor` for structural improvements, extracting shared logic, improving readability or maintainability, renaming, or reorganizing modules when the change does not introduce a feature or primarily fix a bug.

Improvements to existing features, pages, or components should generally be classified as `refactor` when they are structural and behavior-preserving.

Do not classify every improvement as `refactor`. Use `fix` or `perf` when bug correction or performance improvement is the actual primary purpose.

**`chore`**

Use `chore` for dependency upgrades, tooling, build configuration, and routine maintenance unless the repository's established conventions indicate otherwise.

If the repository has documented commit conventions, inspect and follow them.

### 4. Propose a Commit Plan and Wait for Approval

After completing the analysis and grouping, present a commit plan to the user.

For every proposed group, include:

- Group number and proposed commit order
- Change topic
- Proposed commit type
- Proposed scope
- Proposed subject
- Files included
- Summary of the actual changes
- Reason the changes belong together
- Whether further splitting is recommended
- Dependencies on other groups
- Whether partial staging is required
- Relevant risks or unresolved questions

Also identify files that will be excluded, especially `CLAUDE.md`.

Include the branch and remote state from step 1.5 at the top of the plan: the current branch, its upstream, ahead/behind counts, and any warnings.

Use actual diff contents as evidence. Do not invent changes or assume that uninspected files are safe to include.

If any change has unclear intent, explicitly flag it and ask for clarification.

**Stop after presenting the plan. Wait for the user's explicit approval before staging or committing anything.**

Before approval, do not:

- Run `git add`
- Run `git commit`
- Change the Git index or staging state
- Rewrite files to make the proposed groups easier to commit
- Execute other operations that would discard or alter the user's existing changes

Read-only inspection commands are allowed.

### 5. Execute the Approved Plan One Commit at a Time

After receiving explicit approval, follow the approved grouping and commit order.

Before staging the first group, re-run the branch and remote check from step 1.5 (`git fetch --prune` and `git status -sb`). If the branch changed, or the remote gained new commits since the plan was approved, stop and report the difference before committing anything.

For each group:

1. State the change topic being committed.
2. Confirm the final `type(scope): subject`.
3. Recheck that the actual changes match the approved plan.
4. Stage only the changes belonging to the current group.
5. Inspect the staged diff.
6. Verify that unrelated changes and excluded files are not staged.
7. Create the commit using a HEREDOC.
8. Verify that the commit succeeded and inspect its actual contents.
9. Check the remaining Git status before proceeding to the next group.

Do not silently change the approved grouping. If the actual changes differ materially from the plan, stop and obtain approval for the revised plan.

#### Staging Rules

- Prefer explicitly named file paths when staging complete files.
- Never use `git add .` or `git add -A` to stage all changes indiscriminately.
- When a file contains changes belonging to multiple groups, prefer partial staging with `git add -p` or another safe patch-staging method.
- Inspect the staged diff after every staging operation.
- If partial staging cannot be performed safely, do not stage the entire file as a workaround. Explain the limitation and ask the user how to proceed.
- Preserve unrelated staged changes. Do not unstage, reset, or overwrite existing staged content without explicit approval.
- Never stage or commit `CLAUDE.md`.
- Do not include untracked files unless they belong to the approved group.
- If a staged change's ownership or intended destination is unclear, stop and ask.

#### Commit Execution Safety

Do not run the following without explicit user approval:

- `git commit --amend`
- `git reset`
- `git checkout` or `git restore` when they could alter existing changes
- `git push`
- Rebase or other history-rewriting operations

Do not bypass commit hooks with `--no-verify`.

If a commit fails, a hook reports an error, or the resulting commit differs from the approved plan:

1. Stop before committing additional groups.
2. Inspect the failure and current Git state.
3. Preserve all existing changes.
4. Explain the problem and propose a safe next step.
5. Wait for further instructions when resolution requires modifying the approved plan or existing work.

### 6. Report Completion

After all approved commits have been processed, report:

- Number of commits successfully created
- Commit hash for each commit
- Full subject of each commit
- Summary of the changes included in each commit
- Any remaining staged, unstaged, or untracked changes
- Any groups that were skipped or failed
- Final output of `git status --short`
- Final ahead/behind state relative to the upstream (`git status -sb`), so the user knows what a push would involve

Never report a commit as successful unless its creation has been verified.

## Commit Message Convention

Follow Conventional Commits:

```text
<type>(<scope>): <subject>
```

### Header

- `type` is required.
- `scope` is recommended when meaningful and may be omitted when no suitable scope exists.
- `subject` is required.
- Keep the subject concise, specific, and accurate.
- Aim for 50 characters or fewer for the subject.
- Do not end the subject with a period.
- Avoid vague subjects such as `update code`, `fix issue`, or `improve feature`.
- Do not combine unrelated changes in one subject.

### Body

- Keep each line within approximately 72 characters.
- Explain what changed and why.
- Describe relevant behavioral differences when applicable.
- Use bullet points when helpful.
- Ensure every statement matches the actual committed changes.
- Omit the body when a simple change can be described adequately by the header.

### Footer

- Include issue or task references when available and appropriate for the repository.
- Include `BREAKING CHANGE:` only when the change actually introduces a breaking change.
- Omit unnecessary footers.
- **Never include `Co-Authored-By` or `Co-authored-by` trailers, regardless of author identity or tooling defaults.**

## Scope Guidelines

Choose a scope that describes the functional area or change topic rather than mechanically copying a filename.

Examples:

- `auth`
- `product`
- `article`
- `search`
- `upload`
- `api`
- `ui`
- `layout`
- `form`
- `config`
- `router`
- `deps`
- `build`
- `docs`

These are examples, not a mandatory list. Follow the repository's existing naming conventions when they are clear.

## Creating Commit Messages

Use a HEREDOC to preserve multiline commit message formatting.

Example:

```bash
git commit -F - <<'EOF'
refactor(auth): extract shared auth logic

- Extract shared logic into a reusable module
- Preserve existing authentication behavior

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
```

The example is illustrative. Adapt the type, scope, subject, and body to the actual changes.

### Signature Rules

- Do not add a `Co-Authored-By` or `Co-authored-by` trailer.
- Do not add any alternative spelling, capitalization, or formatting of a co-author trailer.
- A Claude Code signature, if used, must remain separate from author or co-author trailers.
- Do not add a signature that the user or repository explicitly prohibits.
- If repository configuration automatically appends a prohibited co-author trailer, identify the configuration and ask before changing it.
- Never modify global Git configuration as part of this skill without explicit approval.

## Final Rules

**Inspect → Check Remote → Analyze → Group → Propose → Wait for Approval → Recheck Remote → Stage → Verify → Commit → Report.**

The user must approve the plan before any staging or commit operation begins.

When there is uncertainty about change ownership, commit boundaries, partial staging, or potential data loss, stop and ask rather than guessing.

The correctness of each commit's contents takes priority over speed or minimizing the number of commits.
