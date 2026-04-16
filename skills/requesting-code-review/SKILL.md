---
name: requesting-code-review
description: Use only when the user explicitly asks to run a code review for completed work, a major feature, or a pre-merge check.
---

# Requesting Code Review

Code review is user-triggered. Do not dispatch a code-reviewer subagent automatically after a task, batch, feature, or implementation session.

Allowed behavior:

- Tell the user that code review is available.
- Wait for an explicit instruction such as “review this,” “run code review,” or “检查代码”。
- After review feedback is returned, fix issues only if the user asks you to address them, unless the user already gave that permission with the review request.

## When to Use

Use when the user explicitly requests code review:

- After completing a task or batch.
- After completing a major feature.
- Before merge or PR.
- When stuck and the user wants a fresh review.

Do not use when:

- The plan merely says a review checkpoint exists.
- A task just completed but the user has not asked for review.
- You are about to move to the next task and want to auto-check the previous one.

## How to Request

1. Get git SHAs:

```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

2. Dispatch code-reviewer subagent only after explicit user instruction.

Use the template at `requesting-code-review/code-reviewer.md`.

Placeholders:

- `{WHAT_WAS_IMPLEMENTED}`：what was built.
- `{PLAN_OR_REQUIREMENTS}`：what it should do.
- `{BASE_SHA}`：starting commit.
- `{HEAD_SHA}`：ending commit.
- `{DESCRIPTION}`：brief summary.

3. Act on feedback according to user instruction:

- Critical issues: report clearly; fix if the user asked you to address review findings.
- Important issues: explain impact before proceeding.
- Minor issues: note for follow-up unless user asks to fix.
- If reviewer is wrong, push back with evidence.

## Integration with Workflows

- `subagent-driven-development` may pause and offer a code review checkpoint, but must not dispatch it without user instruction.
- `executing-plans` may report that a batch is ready for review, but must continue only according to the user's direction.
- `finishing-a-development-branch` may recommend pre-merge code review, but must wait for user approval.

## Red Flags

Never:

- Auto-dispatch code review because a task finished.
- Treat review checkpoints as implicit permission.
- Auto-fix review feedback when the user only asked for a report.
- Proceed with known critical issues without telling the user.

See template at: `requesting-code-review/code-reviewer.md`
