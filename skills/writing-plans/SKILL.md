---
name: writing-plans
description: 在动代码之前，当你已经有一份 spec 或多步骤任务需求时使用
---

# 编写计划

## 概览

编写完整的实现计划，并假设执行这份计划的工程师对我们的代码库几乎没有上下文，而且品味也未必可靠。把他们需要知道的一切都写清楚：每个任务该改哪些文件、可能要查哪些代码/测试/文档、应该怎么测试。把整份计划拆成一口一口能执行的小任务。坚持 DRY、YAGNI、TDD，并频繁提交。

假设对方是熟练开发者，但对我们的工具链和问题领域几乎一无所知。也假设他们并不擅长测试设计。

**开始时声明：** `"I'm using the writing-plans skill to create the implementation plan."`

**上下文：** 这个技能应当在 brainstorming 技能创建的专用 worktree 中运行。

**必需路径子技能：** 在写入或引用任何任务级设计/计划路径前，使用 `superpowers:document-workspace-layout`。优先解析 `DESIGN_DOC_PATH`、`CORE_COGNITION_PATH`、`PLAN_PATH`、`PLAN_CHANGELOG`、`PLAN_WORKSPACE_DIR` 和 `TASK_WORKSPACE_DIR`。

**计划保存位置：** 将规范入口文件保存在解析得到的 `PLAN_WORKSPACE_DIR` 内部的 `PLAN_PATH`
- 正式的实现计划入口文件名由契约固定：`implementation-plan.md`
- 如果计划必须拆分，`PLAN_PATH` 仍然必须是唯一的规范入口/索引文件，旁边再新增顺序分片文件：`implementation-plan-part-01.md`、`implementation-plan-part-02.md`、……
- `planning-with-files-zh` 相关文件必须和 `PLAN_PATH` 以及所有分片文件放在同一个 `PLAN_WORKSPACE_DIR`

## 范围检查

如果 spec 覆盖了多个彼此独立的子系统，那么它本应在 brainstorming 阶段就被拆成多个子项目规格。如果没有拆，你应该建议把它拆成多个计划，每个子系统一个计划。每份计划都必须能独立产出可工作的、可测试的软件。

核心认知文档拆分规则（按设计文档源码行数判断）：
- 如果设计文档 `<=1000` 行，核心认知应保留在设计文档内（不要求单独认知文件）
- 如果设计文档 `>1000` 行，必须有独立的核心认知文档，并将其作为事实来源
- 如果设计文档 `>1000` 行但尚未存在独立核心认知文档，则必须停止写计划，先请求或创建缺失的认知文档

计划编写规则：
- 如果存在独立核心认知文档（或按 `>1000` 规则必须存在），则任务里必须同时引用 spec 和核心认知的相关章节；不要把长篇领域规则/状态定义直接复制进计划
- 如果设计文档 `<=1000` 行，则在任务中引用设计文档中的“核心认知”章节；同样要避免复制冗长背景说明

严格的事实来源对齐规则（强制）：
- 计划内容必须与设计文档和核心认知文档（若无独立认知文档，则与设计文档中的“核心认知”章节）严格对齐。
- 不允许相对源文档新增、删减或语义改写。
- 如果源文档缺少必要事实，就停止计划编写，先更新源文档。
- 每个任务/步骤都必须显式标注其事实来源章节（例如：`Design §x.y`、`Core Cognition §a.b`）。

## 同步纪律（强制）

计划必须始终与其事实来源保持严格同步，确保知识与行动统一：设计文档 +（如适用）独立核心认知文档；若无独立认知文档，则以设计文档中的“核心认知”章节为准。

- 计划内容与源文档之间不允许出现偏差
- 如果源文档变了，必须先更新计划，再继续实现
- 如果实现反馈要求变更，必须先更新设计/核心认知文档，再更新计划
- 任何无法证明与源文档同步的任务，都必须被阻塞，直到重新对齐

## 文件结构

在定义任务之前，先梳理会创建或修改哪些文件，以及每个文件负责什么。这一步会锁定分解决策。

- 设计职责清晰、边界明确的单元。每个文件都应该只有一个清晰职责。
- 你更擅长推理那些一次能放进上下文的代码，而当文件足够聚焦时，你的修改也更可靠。优先选择小而专注的文件，而不是职责过多的大文件。
- 经常一起修改的文件应该放在一起。按职责拆分，而不是按技术层拆分。
- 在现有代码库中，要遵循现有模式。如果代码库本身偏向大文件，不要擅自大规模重构；但如果你要改的文件已经明显失控，那么在计划里加入合理拆分是可以接受的。

这份文件结构会直接影响任务拆分。每个任务都应该形成一组独立成形的改动，单看它本身也有意义。

## 任务粒度：必须是小步快跑

**每一步只做一个动作（2-5 分钟）：**
- “写出失败的测试” 是一步
- “运行它并确认它失败” 是一步
- “写出让测试通过的最小实现” 是一步
- “运行测试并确认通过” 是一步
- “提交” 是一步

## 任务顺序闸门（强制）

每份计划都必须强制串行执行：
- 在 Task 1 之前增加一个全局闸门章节
- 当前任务未标记完成前，不能开始下一个任务
- 每个任务的最后一步必须显式写明“标记当前任务已完成”

计划中必须原样包含以下全局闸门块：

```markdown
### 0. Task Sequencing Gate (Mandatory)

- [ ] After all steps in the current task are done, mark that task as completed (set all checkboxes in that task to `[x]`).
- [ ] Do not start the next task before the current task is marked completed.
```

## 计划文档头部

**`PLAN_PATH` 对应的规范入口文件必须以如下头部开头：**

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

头部校验规则（强制）：
- 源文档路径必须是绝对路径
- 如果独立核心认知文档存在（或按规则必须存在），两个路径都必须声明
- 如果声明缺失，或任一路径不是绝对路径，则停止写计划，先修正 `PLAN_PATH` 的头部
- 如果计划被拆成多个文件，`PLAN_PATH` 必须仍然存在，并且列出所有 part 文件及其执行顺序

## 拆分计划契约

如果单个计划文件足够容纳全部内容，就把完整计划直接写进 `PLAN_PATH`。

如果可执行计划内容会超出文件大小限制：
- `PLAN_PATH` 仍然必须作为唯一的规范入口文件
- 在 `PLAN_PATH` 中保存事实来源声明、目标、架构、技术栈，以及分片文件的有序列表
- 将可执行内容写入按顺序命名的文件：`implementation-plan-part-01.md`、`implementation-plan-part-02.md`、……
- 每个分片文件都应以简短标题开头，例如 `# <Feature Name> Implementation Plan - Part 01`，并回指 `implementation-plan.md`
- 后续执行交接、计划评审引用、以及计划影响更新，仍然必须以 `PLAN_PATH` 为锚点，再更新同一 `PLAN_WORKSPACE_DIR` 内的相关 part 文件

## 任务结构

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

任务步骤标题规则（强制）：
- 与实现相关的步骤标题必须显式要求代码注释
- 注释内容必须在语义上与设计文档和核心认知文档保持一致

## 禁止占位符

每个步骤都必须提供工程师真正需要的实际内容。以下写法都属于 **计划失败**，绝对不要出现：
- `TBD`、`TODO`、`implement later`、`fill in details`
- “Add appropriate error handling” / “add validation” / “handle edge cases”
- “Write tests for the above”（但不给实际测试代码）
- “Similar to Task N”（必须把代码再次写出来，不能假设工程师会按顺序阅读）
- 只描述“要做什么”却不展示“怎么做”的步骤（代码步骤必须有代码块）
- 引用了某些类型、函数、方法，但在任何任务中都没有定义

## 记住
- 永远给出准确文件路径
- 计划里必须放完整代码，不能写成“补上校验”
- 命令必须精确，并写出预期输出
- 需要引用相关技能时，用 `@` 语法
- 能引用认知/规格章节时，就不要重复大段背景
- 坚持 DRY、YAGNI、TDD、频繁提交

## 计划评审循环

每完成一个计划分块后：

1. 派发 plan-document-reviewer 子代理（见 `plan-document-reviewer-prompt.md`），并提供精确构造的评审上下文，绝不要传你的会话历史。这样评审器只看计划本身，而不是你的思考过程。
   - 提供内容：当前分块内容、设计文档绝对路径、核心认知文档绝对路径（如适用）
2. 如果返回 ❌ Issues Found：
   - 修复这个分块中的问题
   - 重新派发评审器审查该分块
   - 重复，直到 ✅ Approved
3. 如果返回 ✅ Approved：继续写下一个分块（或在最后一个分块后进入执行交接）

**分块和文件边界（强制）：**
- 使用 `## Chunk N: <name>` 标题来划定分块
- 每个分块都必须逻辑自洽，且 `<=500` 行
- `PLAN_PATH` 可以是完整计划文件，也可以是计划索引文件
- 每个计划文件都必须 `<=1000` 行
- 如果内容超过 1000 行，必须保留 `PLAN_PATH` 作为索引/入口文件，并把可执行内容拆到 `implementation-plan-part-01.md`、`implementation-plan-part-02.md` 等文件中，同时保持顺序和完整性
- 一个分块必须完整存放在单个文件中，不允许跨文件拆开

**评审循环指导：**
- 由写计划的同一个代理来修复问题（这样上下文最完整）
- 如果循环超过 3 轮，就交给人工指导
- 评审器只提供建议；如果你认为反馈不正确，应明确说明分歧原因
- 只要存在计划/文档漂移（设计 + 核心认知）、分块/文件大小违规，或实现步骤标题没有显式要求代码注释，就必须判定评审失败

## 执行交接

保存计划之后：

**"Plan complete and saved to `<PLAN_PATH>`（若已拆分，按入口文件中的 part 顺序执行）. Ready to execute?"**

**执行路径取决于当前 harness 能力：**

**如果 harness 支持子代理（Claude Code 等）：**
- **必需：** 使用 `superpowers:subagent-driven-development`
- 不要把它当成一个可选项来提供，子代理驱动是标准路径
- 每个任务一个全新子代理，并配合双阶段评审

**如果 harness 不支持子代理：**
- 在当前会话中使用 `superpowers:executing-plans` 执行计划
- 分批执行，并保留检查点供人工评审
