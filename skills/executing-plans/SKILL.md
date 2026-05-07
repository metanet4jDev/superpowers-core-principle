---
name: executing-plans
description: 仅在用户明确要求按现有实现计划执行，或明确要求使用 executing-plans 时使用；正式 review 只在用户明确要求后触发
---

# 执行计划

## 概览

加载计划，检查是否存在阻塞执行的明显缺口，执行全部任务，并在完成后汇报。正式设计/计划/代码 review 不自动执行，只在用户明确要求后触发。
如果计划包含 Code Review Gate，必须停在该门禁：未完成代码评审或未记录人工豁免时，不得宣称编码计划完成。

**限制条件：** 即使当前已经有现成计划，也不能自动使用本技能；只有在用户明确要求按计划执行时才可启用。

**开始时声明：** `"I'm using the executing-plans skill to implement this plan."`
**开始时必需技能：** `"I'm using the document-workspace-layout skill to resolve the plan and code workspaces."`
**开始时必需技能：** `"I'm using the planning-with-files-zh skill to set up and maintain execution tracking files."`

**注意：** 子代理（`superpowers:subagent-driven-development`）仅在用户明确要求时才使用。默认情况下，按本技能流程逐任务串行执行，不主动分派子代理。

## 流程

### 第 0 步：解析任务工作区并启用持久化跟踪
1. 声明：`"I'm using the document-workspace-layout skill to resolve the plan and code workspaces."`
2. **必需技能：** 使用 `document-workspace-layout`
3. 读取或确认 `DOC_WORKSPACE_ROOT`、`TASK_NAME`，以及可选的 `PLAN_NAME`
4. 按 `document-workspace-layout` 的五步规则解析 `PLAN_NAME`，并派生 `PLAN_WORKSPACE_DIR`、`PLAN_PATH`、`PLAN_PART_GLOB`、`TASK_PLAN_PATH`、`FINDINGS_PATH`、`PROGRESS_PATH`、`TASK_WORKSPACE_DIR`
5. 如果 `DOC_WORKSPACE_ROOT`、`TASK_NAME`、`PLAN_NAME` 不满足命名/路径约束，或 `TASK_WORKSPACE_DIR` 不存在，立即停止并向协作者确认
6. 只有在 `document-workspace-layout` 契约允许首次创建时，才创建 `PLAN_WORKSPACE_DIR`；否则若目录不存在，立即停止并确认
7. 切换当前工作目录到 `PLAN_WORKSPACE_DIR`，并将其作为 `planning-with-files-zh` 的唯一写入目录
8. 声明：`"I'm using the planning-with-files-zh skill for persistent tracking."`
9. **必需技能：** 使用 `planning-with-files-zh`
10. 初始化或恢复 `task_plan.md`、`findings.md` 和 `progress.md`；这三个文件必须写入 `PLAN_WORKSPACE_DIR`，也就是与 `PLAN_PATH` 同目录
11. 在开始实现前，确认当前阶段/状态

### 第 1 步：加载计划并检查阻塞项
1. 保持当前工作目录为 `PLAN_WORKSPACE_DIR`
2. 从 `PLAN_PATH` 读取正式计划入口文件；如入口引用 `implementation-plan-part-*.md`，再按顺序读取 `PLAN_PART_GLOB`
3. 同时读取或更新 `task_plan.md`、`findings.md`、`progress.md`，确保持久化跟踪与正式计划同目录
4. 只检查会阻止执行的明显缺口、路径缺失或指令冲突；这不是正式 plan review
5. 如果有疑虑：在开始前先和你的人工协作者讨论
6. 如果没有疑虑：创建 TodoWrite，然后继续

### 第 2 步：执行任务

对于每个任务：
1. 标记为 `in_progress`
2. 执行代码检索、构建、测试、实现相关命令前，切换当前工作目录到 `TASK_WORKSPACE_DIR`
3. 严格按照每一步执行（计划应当已经拆成细粒度步骤）
4. **编码必须使用测试驱动开发（TDD）：** 声明 `"I'm using the test-driven-development skill for coding."`，然后使用 `superpowers:test-driven-development` 技能。先写失败测试，再写最小实现使测试通过，最后重构
5. 按照要求运行验证；验证命令必须在 `TASK_WORKSPACE_DIR` 中执行
6. 需要更新 `task_plan.md`、`findings.md` 或 `progress.md` 时，切回 `PLAN_WORKSPACE_DIR` 再写入
7. 如果当前任务是最终编码任务且计划包含 Code Review Gate，先提醒协作者可以触发代码评审；没有 review 结果或人工豁免记录时，将当前任务标记为 `pending_review` 或 `blocked`，不要标记为 `completed`
8. 如果当前任务不是未满足 Code Review Gate 的最终编码任务，标记为 `completed`

### 第 3 步：完成开发

当所有任务都完成且验证通过后：
- 总结本次实现、验证结果、当前分支状态和遗留风险
- 如果 Code Review Gate 已完成或已有人工豁免，告诉用户当前工作已经进入可收尾状态
- 如果 Code Review Gate 未完成，告诉用户代码和验证已到评审门禁，等待 review 或人工豁免后才能完成编码计划
- 只有在用户明确要求收尾、合并、开 PR、保留分支或丢弃工作时，才使用 `superpowers:finishing-a-development-branch`

## 何时停止并求助

**出现以下情况时，必须立即停止执行：**
- 遇到阻塞（依赖缺失、测试失败、指令不清楚）
- 计划存在关键缺口，导致无法开工
- 你无法理解某条指令
- 验证重复失败

**不要猜，先澄清。**

## 何时回到前面的步骤

**以下情况需要回到“加载计划并检查阻塞项”（第 1 步）：**
- 协作者根据你的反馈更新了计划
- 基本实现路径需要重新思考

**不要硬顶着阻塞往前冲。**

## 记住
- 先检查计划是否存在阻塞执行的问题；不要自动做正式 review
- 严格按计划执行
- 不要跳过验证
- 计划要求引用技能时就必须引用
- 遇到阻塞就停下，不要猜
- 未经用户明确同意，绝不要在 `main/master` 分支上直接开始实现
- 不要把文档/计划目录当成代码目录；计划和 tracking 文件只写入 `PLAN_WORKSPACE_DIR`，代码命令只在 `TASK_WORKSPACE_DIR` 中执行

## 集成

**必需工作流技能：**
- **document-workspace-layout**：必需。用于先解析 `DOC_WORKSPACE_ROOT`、`TASK_NAME`、可选 `PLAN_NAME`，确定 `PLAN_WORKSPACE_DIR`、`PLAN_PATH`、三件套写入目录和 `TASK_WORKSPACE_DIR`
- **planning-with-files-zh**：必需。用于在工具调用和跨会话中持久化 plan/progress/findings 状态（位于 `/home/haodev/.agents/skills/planning-with-files-zh`）
