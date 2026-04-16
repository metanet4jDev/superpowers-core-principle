---
name: executing-plans
description: 当你已经有一份书面的实现计划，并需要在独立会话中执行它时使用；正式 review 只在用户明确要求后触发
---

# 执行计划

## 概览

加载计划，检查是否存在阻塞执行的明显缺口，执行全部任务，并在完成后汇报。正式设计/计划/代码 review 不自动执行，只在用户明确要求后触发。

**开始时声明：** `"I'm using the executing-plans skill to implement this plan."`
**开始时必需技能：** `"I'm using the planning-with-files-zh skill to set up and maintain execution tracking files."`

**注意：** 子代理（`superpowers:subagent-driven-development`）仅在用户明确要求时才使用。默认情况下，按本技能流程逐任务串行执行，不主动分派子代理。

## 流程

### 第 0 步：启用持久化跟踪
1. 声明：`"I'm using the planning-with-files-zh skill for persistent tracking."`
2. **必需技能：** 使用 `planning-with-files-zh`
3. 初始化或恢复 `task_plan.md`、`findings.md` 和 `progress.md`
4. 在开始实现前，确认当前阶段/状态

### 第 1 步：加载计划并检查阻塞项
1. 读取计划文件
2. 只检查会阻止执行的明显缺口、路径缺失或指令冲突；这不是正式 plan review
3. 如果有疑虑：在开始前先和你的人工协作者讨论
4. 如果没有疑虑：创建 TodoWrite，然后继续

### 第 2 步：执行任务

对于每个任务：
1. 标记为 `in_progress`
2. 严格按照每一步执行（计划应当已经拆成细粒度步骤）
3. 按照要求运行验证
4. 标记为 `completed`

### 第 3 步：完成开发

当所有任务都完成且验证通过后：
- 声明：`"I'm using the finishing-a-development-branch skill to complete this work."`
- **必需子技能：** 使用 `superpowers:finishing-a-development-branch`
- 按该技能要求完成测试验证、展示选项并执行最终选择

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

## 集成

**必需工作流技能：**
- **planning-with-files-zh**：必需。用于在工具调用和跨会话中持久化 plan/progress/findings 状态（位于 `/home/haodev/.agents/skills/planning-with-files-zh`）
- **superpowers:using-git-worktrees**：必需。用于在开始前建立隔离工作区
- **superpowers:writing-plans**：用于创建本技能要执行的计划
- **superpowers:finishing-a-development-branch**：用于在开发完成后收尾
