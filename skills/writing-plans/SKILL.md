---
name: writing-plans
description: 仅在用户明确要求编写实现计划、implementation plan，或明确要求使用 writing-plans 时使用。
---

# 编写实现计划

## Overview

本技能从已批准的详细设计事实源生成实现计划。计划必须足够具体，让一个熟练但不了解代码库和业务背景的工程师也能按步骤完成实现、测试和提交。

**限制条件：** 只有在用户明确要求编写计划时才能使用本技能；即使看起来“现在应该先写计划”，也不能自动触发。

开始时声明：

> "I'm using the writing-plans skill to create the implementation plan."

核心原则：

- 计划只消费设计事实，不新增、删除或语义改写设计。
- 计划 review 不能自动执行；只能在用户明确要求后触发。
- 计划任务必须小步快跑，默认每步 2-5 分钟。
- 坚持 DRY、YAGNI、TDD 和频繁提交。
- 每个任务都要给出准确文件路径、实际代码/命令、预期结果和验证方式。
- 如果事实源缺失或与计划不一致，先回到 `writing-detailed-design` 修设计，再写计划。

## Mandatory Path Contract

写入或引用任务级路径前，必须先用 `superpowers:document-workspace-layout` 解析：

- `DESIGN_DOC_PATH`
- `CORE_COGNITION_PATH`
- `SPECS_DIR`
- `DESIGN_ARTIFACTS_DIR`
- `PLAN_PATH`
- `PLAN_CHANGELOG`
- `PLAN_WORKSPACE_DIR`
- `TASK_WORKSPACE_DIR`

计划保存位置：

- `PLAN_PATH` 是唯一正式入口，文件名固定为 `implementation-plan.md`。
- 如果计划拆分，入口仍是 `PLAN_PATH`，分片只能放在同一 `PLAN_WORKSPACE_DIR` 下：`implementation-plan-part-01.md`、`implementation-plan-part-02.md`。
- `planning-with-files-zh` 的 `task_plan.md`、`findings.md`、`progress.md` 必须和 `PLAN_PATH` 同目录。

## Source of Truth Gate

写计划前必须确认：

- 设计文档已经存在并被批准。
- 独立 `CORE_COGNITION_PATH` 已存在，并被设计文档引用。
- 配套文件被主文档引用，且必要内容已存在。
- 当前计划不会相对设计文档新增、删减或语义改写。

严格规则：

- 每个任务或步骤都必须显式标注事实来源章节，例如 `Design §6.2`、`Core Cognition §3`。
- 如果源文档缺少必要事实，停止写计划，先更新设计。
- 如果实现反馈要求改变行为、状态、接口或存储，先更新设计/核心认知，再更新计划。
- 任何无法证明与源文档同步的任务都必须阻塞。

## Scope Check

如果 spec 覆盖多个彼此独立的子系统，而还没有拆成多个设计或计划，应停止并建议拆分。每份计划必须能独立产出可工作、可测试的软件。

检查：

- 是否跨多个独立子系统。
- 是否需要多个发布或迁移阶段。
- 是否存在无法在单份计划中串行完成的工作流。
- 是否已有 plan drift，需要先同步旧计划。

## File Structure First

定义任务前，先写文件结构：

- 会创建哪些文件。
- 会修改哪些文件。
- 每个文件负责什么。
- 哪些测试文件覆盖哪些行为。
- 哪些配套设计文件是事实来源。

文件职责要清晰。经常一起修改的文件可以放在一起；按职责拆分，不按技术层机械拆分。在现有代码库中遵循现有模式，不做无关重构。

## Task Sequencing Gate

计划中必须原样包含以下全局闸门块：

```markdown
### 0. Task Sequencing Gate (Mandatory)

- [ ] After all steps in the current task are done, mark that task as completed (set all checkboxes in that task to `[x]`).
- [ ] Do not start the next task before the current task is marked completed.
```

每个任务最后一步必须写明“标记当前任务已完成”。

## Plan Header

`PLAN_PATH` 对应的入口文件必须以如下头部开头：

```markdown
# [Feature Name] Implementation Plan

> **Source of Truth Declaration (Mandatory)**
> This plan is written strictly according to the design/core-cognition source documents, with no additions, deletions, or semantic modifications.
>
> Design Document (absolute path): `/ABS/PATH/TO/design-codex.md`
> Core Cognition Document (absolute path): `/ABS/PATH/TO/core-cognition.md`
>
> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

头部校验规则：

- 源文档路径必须是绝对路径。
- 必须声明独立核心认知文档的绝对路径。
- 如果计划拆分，入口文件必须列出所有 part 文件及执行顺序。

## Task Template

每个任务都必须形成一组独立成形的改动。与实现相关的步骤标题必须显式要求代码注释，且注释语义必须与设计/核心认知一致。

````markdown
### Task N: [Component Name]

**Source:** Design §x.y; Core Cognition §a.b

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

- [ ] **Step 3: Write minimal implementation with required code comments aligned to Design §x.y and Core Cognition §a.b**

```python
def function(input):
    # Must match Design §x.y and Core Cognition §a.b.
    return expected
```

- [ ] **Step 4: Verify comments and behavior match the source documents, then run test to verify it passes**

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

## Plan Splitting

如果单个计划文件足够容纳全部内容，就把完整计划写进 `PLAN_PATH`。

如果可执行内容会超过文件大小限制：

- `PLAN_PATH` 仍是唯一入口。
- `PLAN_PATH` 保存事实来源声明、目标、架构、技术栈和分片顺序。
- 可执行内容写入 `implementation-plan-part-01.md`、`implementation-plan-part-02.md`。
- 每个分片文件以简短标题开头，并回指 `implementation-plan.md`。
- 每个分块必须完整存放在单个文件中，不跨文件拆开。

文件和分块限制：

- 每个计划文件 `<=1000` 行。
- 每个 `## Chunk N: <name>` 分块 `<=500` 行。

## Forbidden Placeholders

计划中不得出现：

- `TBD`、`TODO`、`implement later`、`fill in details`。
- “Add appropriate error handling” 但不给具体错误语义。
- “add validation” 但不给具体字段、条件、预期错误。
- “Write tests for the above” 但不给实际测试代码。
- “Similar to Task N”。
- 只写“要做什么”，不写“怎么做”。
- 引用类型、函数、方法，但任何任务都没有定义它们。

## User-Triggered Plan Review

计划 review 不自动执行。每完成一个计划文件或计划分块后，只能提示用户：

> "Plan saved. Tell me if you want me to run a plan review."

只有用户明确要求“review / 审查 / 检查计划”后，才使用 `plan-document-reviewer-prompt.md` 派发 plan-document-reviewer。

用户触发计划审查时：

1. 提供当前分块、`DESIGN_DOC_PATH`、`CORE_COGNITION_PATH` 和配套文件列表；不要传会话历史。
2. 如果返回 `Issues Found`，按用户要求修改；修改后是否复审，也必须等待用户明确指令。
3. 如果超过 3 轮仍未通过，交给人工指导。

只要存在计划/设计漂移、事实来源缺失、分块大小违规、任务步骤不可执行，或实现步骤标题没有显式要求代码注释，就必须判定审查失败。

## Handoff

保存计划后输出：

> "Plan complete and saved to `<PLAN_PATH>`（若已拆分，按入口文件中的 part 顺序执行）. Ready to execute?"

执行路径：

- 如果 harness 支持子代理，使用 `superpowers:subagent-driven-development`。
- 如果 harness 不支持子代理，使用 `superpowers:executing-plans` 分批执行并保留人工检查点。

不要在 `writing-plans` 阶段直接实现。

## Completion Check

完成计划前确认：

- `PLAN_PATH` 存在，且是唯一入口。
- 事实来源路径都是绝对路径。
- 每个任务都有来源章节、文件范围、测试、命令、预期结果和提交步骤。
- 每个实现步骤都要求注释与设计/核心认知对齐。
- 没有占位符和语义漂移。
- 计划足以交给不了解背景的工程师执行。
