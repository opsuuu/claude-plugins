---
name: handoff
description: Use when a stage of a multi-stage plan is finished and work must be handed off to a new session, or when the user types /handoff or asks for a handoff. Compacts the current conversation into a handoff file plus a short paste-ready prompt for the next session.
argument-hint: "What will the next session be used for?"
disable-model-invocation: true
---

# Handoff

Goal: a brand-new session, with none of the current conversation's context, should be able to pick up the work by reading a single file.

Write the handoff file and the paste-ready prompt in Traditional Chinese (keep code and technical terms in English).

## Arguments

User arguments: $ARGUMENTS

- With arguments: they describe the next session's focus. Keep details that serve that focus; cut or shorten everything else.
- Without arguments: the focus is the next stage of the plan.

## Steps

### 1. Determine basic info

Get these with commands; do not guess.

- Repo root: `git rev-parse --show-toplevel`
- Project name: the repo root folder name
  `basename "$(git rev-parse --show-toplevel)"`
- Date: `date +%Y%m%d`
- Stage name: use the stage name from the plan (e.g. `stage2-permission-api`). If unsure, ask the user.

### 2. Collect the actual state of the work

Use git as the source of truth, not conversation memory.

- `git status`: uncommitted changes
- `git diff --stat`: changed files and size of changes
- `git log --oneline`: commits made during this stage, if any

Also note any existing artifacts related to this work: plans, PRDs, ADRs, issues, pull/merge requests, earlier handoff files.

### 3. Write the handoff file

- Path: `<repo root>/.handoff/handoff-<YYYYMMDD>-<stage>.md`
  - The handoff file is for the next session to read, not for the user, so it stays inside the project and is never pushed.
  - If the `.handoff/` folder does not exist, create it with `mkdir -p`.
  - Before writing, run `git check-ignore -q .handoff/` from the repo root and keep the result for step 4. Keep the trailing slash: without it, the check reports "not ignored" while the folder does not exist yet. Write the file whatever the result is.
- If a file with the same name already exists, do not overwrite it on your own. Show the user the existing file's path and its header (date, next-session focus), then ask which to do:
  1. Append `-2`, `-3`, etc. to the new filename
  2. Use a different stage name the user provides
  3. Overwrite the existing file (it is outdated)
  4. Keep the existing file, write nothing new, and only output the paste-ready prompt for it
- Follow the "Handoff file template" below

Writing rules:

- Write for someone who has never seen this conversation. Avoid references like "the thing above" or "what we just did".
- Treat the handoff file as an index plus the missing pieces: link existing records (plans, PRDs, ADRs, issues, commits, diffs) by path, commit hash, or URL, and spell out only what exists nowhere else, such as decisions made in conversation, pitfalls, and open questions.
- Give file paths relative to the repo root.
- For decisions, record why, not just what, so the next session does not reopen settled questions.
- Record only facts and confirmed information. Put anything speculative or unverified in the「待查證事項」section, together with how to verify it; do not present it as fact in other sections.
- Redact sensitive information: API keys, tokens, passwords, connection strings, and personally identifiable information (names, phone numbers, emails, ID numbers of real people in data). Describe them generically instead (e.g. "API key is in `.env` as `XXX_API_KEY`").

### 4. Make sure `.handoff/` is ignored

Skip this step if the check in step 3 showed that `.handoff/` is already ignored.

Otherwise, tell the user that `.handoff/` is not ignored, so the handoff file shows up as an untracked file and could be committed by mistake. Ask whether to add `.handoff/` to their global gitignore now.

- **Yes**:
  1. Run `git config --global core.excludesFile` to find the global gitignore. If there is no output, use git's default location: `$XDG_CONFIG_HOME/git/ignore`, or `~/.config/git/ignore` when `XDG_CONFIG_HOME` is not set. Do not set `core.excludesFile`.
  2. If the file already has a `.handoff/` line, do not add it again. Otherwise create the parent folder if needed and append `.handoff/` on its own line.
  3. Re-run `git check-ignore -q .handoff/` and report the result. If it is still not ignored (for example, the project's `.gitignore` re-includes it with `!.handoff/`), explain why and leave the project's `.gitignore` unchanged.
- **No**: Save a note to your persistent memory saying that `.handoff/` is not yet in the global gitignore on this machine, so a later session can remind the user. Do not save a duplicate if such a note already exists. If no persistent memory is available, tell the user the note could not be saved. Then remind the user to set it up later.

Only ever edit the global gitignore, and only after the user agrees. Never edit a project's `.gitignore`.

### 5. Output the paste-ready prompt

In the conversation, provide a short prompt inside a code block that the user can copy into the new session. It contains only:

1. The absolute path of the handoff file (`<repo root>/.handoff/...`), so it still resolves when the new session starts in a subdirectory or another worktree
2. An instruction for the new session to read that file first
3. An instruction to verify the items in「待查證事項」before relying on them, instead of treating them as fact (skip this if the section is「無」)
4. What the next session should do, in one sentence

Do not repeat the handoff file's content. Example:

```
請先讀取 /Users/me/projects/my-app/.handoff/handoff-20261028-stage2-permission-api.md 了解目前進度與決策，「待查證事項」中的內容請先查證再使用，接著進行階段 3：權限管理頁面 UI 串接。
```

### 6. Remind the user to start a new session

Finally, remind the user to open a new session and paste the prompt. Do not continue with the next stage in the current session.

## Handoff file template

```markdown
# 交接：<project> — <stage>

- 日期：<YYYY-MM-DD>
- 分支：<current branch>
- 未 commit 的改動：<有／無；有的話列出>
- 下一個 session 的重點：<from arguments, or "繼續下一階段">

## 整體目標與目前進度

<What the whole plan aims to achieve; how many stages in total; where things stand now>

## 相關文件與紀錄

- <plan / PRD / ADR / issue / pull/merge request / earlier handoff>：<path or URL>
- 本階段 commit：<hash range, or "尚未 commit">

## 本階段完成了什麼

- <completed item; reference commits instead of restating diffs>

### 改動的檔案

- `<path>`：<what changed, one line>

## 下一階段要做什麼

- <concrete task; the more specific the better>
- <suggested starting point: which file, which function>

## 已做的決策與原因

- <decision>：<reason; alternatives considered and why they were rejected>

## 待查證事項

- <unverified item>：<how to verify: which doc, command, or file to check>
- <if there are none, write「無」>

## 已知的坑與未解問題

- <pitfalls hit, things to watch out for>
- <open questions that need the user's decision>

## 建議使用的 skills

- `<skill name>`：<when and why the next session should invoke it>
```
