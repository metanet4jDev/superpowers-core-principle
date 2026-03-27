---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

Core cognition doc split rule (by design doc source line count):
- If design doc `<=1000` lines, core cognition should stay inside the design doc (no separate cognition file required)
- If design doc `>1000` lines, a separate core cognition doc is required and treated as the fact source
- If design doc `>1000` lines but no separate core cognition doc exists yet, stop plan writing and request/create the missing cognition doc first

Plan writing rule:
- If separate core cognition doc exists (or is required by the `>1000` rule), reference both spec and core cognition sections in tasks; do not copy long domain rules/state definitions into the plan
- If design doc `<=1000` lines, reference the design doc's "核心认知" section in tasks; still avoid duplicating long background text

Strict source-of-truth alignment rule (mandatory):
- Plan content must remain strictly aligned with the design document and core cognition document (or the design doc's "核心认知" section when no separate cognition file exists).
- No additions, deletions, or semantic rewrites are allowed relative to source documents.
- If required facts are missing from source documents, stop planning and update the source documents first.
- Every task/step must include explicit section references to its source of truth (for example: `Design §x.y`, `Core Cognition §a.b`).

## Sync Discipline (Mandatory)

The plan must stay strictly synchronized with its source of truth at all times, ensuring unity of knowledge and action ("知行合一"): design document + separate core cognition document when applicable, or the design document's "核心认知" section otherwise.

- No divergence is allowed between plan content and source documents.
- If source documents change, update the plan immediately before continuing implementation.
- If implementation feedback requires change, update design/core-cognition docs first, then update the plan.
- Any task that cannot prove sync with source documents must be blocked until alignment is restored.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Task Sequencing Gate (Mandatory)

Every plan must enforce strict sequential execution:
- Add a global gate section before Task 1
- The next task cannot start until the current task is marked completed
- "Mark current task completed" must be written as an explicit final step inside each task

Use this exact global gate block in plans:

```markdown
### 0. Task Sequencing Gate (Mandatory)

- [ ] After all steps in the current task are done, mark that task as completed (set all checkboxes in that task to `[x]`).
- [ ] Do not start the next task before the current task is marked completed.
```

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **Source of Truth Declaration (Mandatory)**
> This plan is written strictly according to the design/core-cognition source documents, with no additions, deletions, or semantic modifications.
>
> Design Document (absolute path): `/ABS/PATH/TO/design-doc.md`
> Core Cognition Document (absolute path): `/ABS/PATH/TO/core-cognition.md` (if applicable)
>
> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

Header validation rules (mandatory):
- Source document paths must be absolute paths.
- If a separate core cognition document exists (or is required by rule), both paths must be declared.
- If declaration is missing or any path is not absolute, stop plan writing and fix the header first.

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation with required code comments aligned to Design/Core Cognition**

```python
def function(input):
    # This behavior must match Design §x.y and Core Cognition §a.b
    return expected
```

- [ ] **Step 4: Verify comments and behavior both match Design/Core Cognition, then run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```

- [ ] **Step 6: Mark Task N as completed, then move to Task N+1**

Set all checkboxes in this task to `[x]`. Do not start Task N+1 before this is done.
````

Task step-title rule (mandatory):
- Implementation-related step titles must explicitly require code comments.
- Comment content must stay semantically consistent with the design document and core cognition document.

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- Prefer section references over duplicated prose across cognition/spec/plan
- DRY, YAGNI, TDD, frequent commits

## Plan Review Loop

After completing each chunk of the plan:

1. Dispatch plan-document-reviewer subagent (see plan-document-reviewer-prompt.md) with precisely crafted review context — never your session history. This keeps the reviewer focused on the plan, not your thought process.
   - Provide: chunk content, absolute path to design document, absolute path to core cognition document (if applicable)
2. If ❌ Issues Found:
   - Fix the issues in the chunk
   - Re-dispatch reviewer for that chunk
   - Repeat until ✅ Approved
3. If ✅ Approved: proceed to next chunk (or execution handoff if last chunk)

**Chunk and file boundaries (mandatory):**
- Use `## Chunk N: <name>` headings to delimit chunks.
- Each chunk must be logically self-contained and must be `<=500` lines.
- Each plan file must be `<=1000` lines.
- If content exceeds 1000 lines, split into sequential files (for example: `...-part-01.md`, `...-part-02.md`), preserving order and completeness.
- A chunk must be stored entirely in a single file; cross-file chunk storage is not allowed.

**Review loop guidance:**
- Same agent that wrote the plan fixes it (preserves context)
- If loop exceeds 3 iterations, surface to human for guidance
- Reviewers are advisory - explain disagreements if you believe feedback is incorrect
- Fail review if any plan/doc drift exists (design + core cognition), if chunk/file size rules are violated, or if implementation step titles omit explicit code-comment requirements.

## Execution Handoff

After saving the plan:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Ready to execute?"**

**Execution path depends on harness capabilities:**

**If harness has subagents (Claude Code, etc.):**
- **REQUIRED:** Use superpowers:subagent-driven-development
- Do NOT offer a choice - subagent-driven is the standard approach
- Fresh subagent per task + two-stage review

**If harness does NOT have subagents:**
- Execute plan in current session using superpowers:executing-plans
- Batch execution with checkpoints for review
