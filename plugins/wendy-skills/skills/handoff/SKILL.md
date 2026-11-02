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

If arguments were given, treat them as a description of what the next session will focus on, and tailor the handoff accordingly: prioritize context relevant to that focus and trim what is not. If no arguments were given, assume the next session continues with the next stage of the plan.

## Steps

### 1. Determine basic info

Get these with commands; do not guess.

- Project name: the repo root folder name
  `basename "$(git rev-parse --show-toplevel)"`
- Date: `date +%Y%m%d`
- Stage name: use the stage name from the plan (e.g. `stage2-permission-api`). If unsure, ask the user.

### 2. Collect the actual state of the work

Use git as the source of truth, not conversation memory.

- `git status`: uncommitted changes
- `git diff --stat`: changed files and size of changes
- `git log --oneline`: commits made during this stage, if any

Also note any existing artifacts related to this work: plans, PRDs, ADRs, issues, MRs, earlier handoff files.

### 3. Write the handoff file

- Path: `/Users/wendylai/Desktop/CLAUDE_DECISION/<project>/handoff-<YYYYMMDD>-<stage>.md`
- If the folder does not exist, create it with `mkdir -p`
- If a file with the same name exists, do not overwrite it; append `-2`, `-3`, etc. to the filename
- Follow the "Handoff file template" below

Writing rules:

- Write for someone who has never seen this conversation. Avoid references like "the thing above" or "what we just did".
- Do not duplicate content already captured in other artifacts (plans, PRDs, ADRs, issues, commits, diffs). Reference them by path, commit hash, or URL instead, and summarize only what the next session needs to know.
- Give file paths relative to the repo root.
- For decisions, record why, not just what, so the next session does not reopen settled questions.
- Record only facts and confirmed information. Clearly mark anything speculative or unverified.
- Redact sensitive information: API keys, tokens, passwords, connection strings, and personally identifiable information (names, phone numbers, emails, ID numbers of real people in data). Describe them generically instead (e.g. "API key is in `.env` as `XXX_API_KEY`").

### 4. Output the paste-ready prompt

In the conversation, provide a short prompt inside a code block that the user can copy into the new session. It contains only:

1. The full path of the handoff file
2. An instruction for the new session to read that file first
3. What the next session should do, in one sentence

Do not repeat the handoff file's content. Example:

```
請先讀取 /Users/wendylai/Desktop/CLAUDE_DECISION/vip-platform/handoff-20261028-stage2-permission-api.md 了解目前進度與決策，接著進行階段 3：權限管理頁面 UI 串接。
```

### 5. Remind the user to start a new session

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

- <plan / PRD / ADR / issue / MR / earlier handoff>：<path or URL>
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

## 已知的坑與未解問題

- <pitfalls hit, things to watch out for>
- <open questions that need the user's decision>

## 建議使用的 skills

- `<skill name>`：<when and why the next session should invoke it>
```
