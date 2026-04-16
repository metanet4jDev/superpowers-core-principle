---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session.
---

# Subagent-Driven Development

Execute an implementation plan by dispatching fresh implementer subagents for independent tasks. Formal spec review, plan review, and code review are user-triggered only.

Core principles:

- Fresh implementer subagent per task.
- Controller provides exact task text and context; subagents do not read the whole plan by default.
- Subagents verify their own implementation with tests and commands required by the plan.
- Formal review checkpoints are offered, not automatically executed.

## Review Trigger Rule

Design review, plan review, spec compliance review, and code quality review must not run automatically.

Allowed:

- “Task N is complete. Tell me if you want me to run spec or code review before continuing.”
- Continue only according to the user's instruction.

Forbidden:

- Dispatching spec reviewer after every task without user instruction.
- Dispatching code quality reviewer after every task without user instruction.
- Dispatching final code reviewer automatically at the end.
- Re-reviewing after fixes unless the user explicitly asks.

## When to Use

Use when:

- There is an implementation plan.
- Tasks are mostly independent.
- You can stay in the current session as coordinator.
- The user wants subagent-driven execution or the harness supports it and the plan allows it.

Use `executing-plans` instead when:

- Tasks are tightly coupled.
- The environment does not support subagents.
- The user wants single-session execution.

## Process

1. Read the plan entry and extract all tasks with full task text.
2. Create a task tracker.
3. For each task:
   - Dispatch one implementer subagent with full task text, relevant context, working directory, source references, and verification commands.
   - Answer implementer questions.
   - Wait for implementer result.
   - Inspect status, changed files, commits, and test results enough to coordinate safely.
   - If implementation completed, offer a user-triggered review checkpoint.
   - Move to the next task only if the plan sequencing gate and user direction allow it.
4. After all tasks complete, summarize changed files, verification, commits, and unresolved concerns.
5. Offer final code review as an option, but do not run it automatically.
6. When user chooses to finish, use `superpowers:finishing-a-development-branch`.

## Implementer Status

Implementer subagents report one of four statuses:

- `DONE`：task completed and required verification passed.
- `DONE_WITH_CONCERNS`：task completed, but there are doubts or risks.
- `NEEDS_CONTEXT`：subagent needs more information.
- `BLOCKED`：subagent cannot complete the task.

Handling:

- `DONE`：record result, offer review checkpoint if useful, then follow user direction.
- `DONE_WITH_CONCERNS`：read concerns and decide whether to clarify, fix, or ask user before continuing.
- `NEEDS_CONTEXT`：provide missing context and redispatch.
- `BLOCKED`：diagnose whether context, plan, model capability, or task size is the issue; do not retry unchanged.

## Model Selection

Use the least powerful model that can handle the role:

- Mechanical isolated tasks: fast, cheap model.
- Multi-file integration or debugging: standard model.
- Architecture judgment or explicit review requested by the user: most capable available model.

## Prompt Templates

- `./implementer-prompt.md`：implementer subagent.
- `./spec-reviewer-prompt.md`：user-triggered spec compliance review.
- `./code-quality-reviewer-prompt.md`：user-triggered code quality review.

## Example Workflow

```text
You: I'm using Subagent-Driven Development to execute this plan.

[Read plan, extract Task 1, dispatch implementer.]

Implementer:
  Status: DONE
  Files changed: ...
  Tests: passing
  Concerns: none

You:
  Task 1 is implemented and tests passed.
  Tell me if you want spec review or code review before I continue to Task 2.

User:
  Continue.

[Proceed to Task 2.]
```

If the user says “run code review,” use `requesting-code-review`. If the user says “run spec compliance review,” use `spec-reviewer-prompt.md`. After review findings, ask or follow the user's explicit instruction before applying fixes.

## Red Flags

Never:

- Start implementation on main/master without explicit user consent.
- Dispatch multiple implementation subagents that edit overlapping files.
- Make implementer subagents read the full plan when the controller can provide task text.
- Ignore implementer questions.
- Treat review checkpoints as permission to review.
- Auto-fix review feedback when the user only asked for a report.
- Move to the next task if the plan sequencing gate forbids it.

## Integration

Required workflow skills:

- `superpowers:using-git-worktrees`：set up isolated workspace before starting, unless the user explicitly chose a different safe workflow.
- `superpowers:writing-plans`：creates the plan this skill executes.
- `superpowers:requesting-code-review`：only when the user asks for code review.
- `superpowers:finishing-a-development-branch`：complete development after all tasks.

Subagents should use:

- `superpowers:test-driven-development` when the task changes behavior and the plan requires TDD.

Alternative workflow:

- `superpowers:executing-plans` for single-session execution.
