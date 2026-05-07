---
name: brainstorming
description: "Use before implementation to clarify a feature, system, component, architecture, or behavior change; includes existing-code exploration when architecture or module understanding affects the design."
---

# 将想法打磨成可批准的设计

通过协作式对话，把粗略想法收敛成可批准的方案。`brainstorming` 负责澄清目标、建立核心认知、比较方案、取得用户批准；详细设计文档由 `writing-detailed-design` 负责；实现计划由 `writing-plans` 负责。

## Mandatory Skills

- 先用 `superpowers:core-principle` 建立 `V0` 认知骨架。
- 在探索代码、澄清需求、比较方案时，只要出现新证据、新约束或冲突结论，就把核心认知更新为 `V1..Vn`。
- 写入或引用任何任务级设计/计划路径前，先用 `superpowers:document-workspace-layout` 解析 `DESIGN_DOC_PATH`、`CORE_COGNITION_PATH`、`DESIGN_ARTIFACTS_DIR`、`PLAN_PATH`、`PLAN_CHANGELOG`、`PLAN_WORKSPACE_DIR` 和 `TASK_WORKSPACE_DIR`。
- 设计被批准后，进入详细设计前，必须先完成 Foundation Gate：基于真实代码库、真实数据库环境和真实依赖环境，按功能模块列出 API 清单、时序图清单、DDL/存储约束和数据一致性方案，并等待人工确认。
- Foundation Gate 人工确认后，调用 `superpowers:writing-detailed-design` 写入或更新详细设计产物；详细设计只引用独立核心认知文档。
- 设计产物检查、计划 review、代码 review 都不能自动触发；只能在用户明确要求后执行。
- 详细设计完成并经用户确认后，才能调用 `superpowers:writing-plans`。

<HARD-GATE>
在已经展示设计且用户明确批准之前，不要调用实现技能、不要写代码、不要搭建项目、不要执行实现动作。无论任务看起来多简单，这条规则都适用。
</HARD-GATE>

## When to Use

Use when:

- 用户提出新功能、新系统、新组件、行为变更或架构调整。
- 当前需求仍有目标、范围、边界、状态或兼容性不清楚。
- 需要在多个方案之间权衡。
- 现有代码库结构会影响设计。

Do not use when:

- 用户只要求执行一条确定命令。
- 已有批准的详细设计和实现计划，当前只是在执行计划。
- 用户明确只要求做设计产物检查；这种场景使用 `reviewing-design-artifacts`。

## Workflow

1. **建立核心认知 V0**
   按 `core-principle` 的 7 维顺序输出最小骨架，不能跳过或整维留空。缺失信息标 `待确认`。

2. **探索上下文**
   如果已有代码库，先解析 `TASK_WORKSPACE_DIR`，再查看文件、文档、测试、最近提交和现有模式。把证据回填到 7 维认知中。

3. **必要时提供视觉伴侣**
   如果接下来的问题涉及 mockup、布局、图表或视觉比较，按“视觉伴侣”章节单独征求同意。

4. **一次只问一个澄清问题**
   聚焦目标、约束、成功标准、关键属性、状态、异常和兼容性。能用多选就用多选。

5. **持续更新核心认知**
   新信息进入后，先更新对应维度，再继续提问或方案比较。

6. **提出 2-3 个方案**
   先讲推荐方案，再讲备选。每个方案至少说明适用条件、额外复杂度、行为差异、兼容影响。

7. **分节展示设计并取得批准**
   覆盖架构、组件、数据流、状态、错误处理、测试和风险。复杂设计分节确认；简单设计可以很短，但仍需明确批准。

8. **完成 Foundation Gate**
   用户批准方案后，先确保 `CORE_COGNITION_PATH` 保存 `Vfinal`，再在 `DESIGN_ARTIFACTS_DIR/foundation-gate.md` 按功能模块列出地基产物：真实依据、模块边界、功能点、API、主时序图、DDL/存储约束和数据一致性方案。API、主时序图和 DDL 必须基于真实代码库、真实数据库环境和真实依赖环境。状态不是 `confirmed` 时，不得进入详细设计。

9. **进入详细设计**
   Foundation Gate 人工确认后，调用 `writing-detailed-design` 写 `design-codex.md` 和配套产物。可以提示用户“如需整体检查，可明确要求触发”，但不能自动执行检查。

10. **进入计划**
   详细设计完成并经用户确认后，调用 `writing-plans`。不要直接进入实现。

## Core-Cognition Discipline

在整个 brainstorming 中，`core-principle` 不是开场白，而是持续主线：

- `V0`：讨论前的最小认知骨架。
- `V1..Vn`：每次新证据、新约束、冲突结论后的增量更新。
- `Vfinal`：方案批准前，用于支撑详细设计和测试的稳定认知。

始终保持：

- 7 维顺序固定：结构、分类、关系、属性、状态、功能、界限。
- `界限` 必须是可用条件和执行后果，不是泛泛的架构边界。
- 关键属性必须说明类型、唯一性、可编辑性、必填、长度/精度；影响判断但未知时，优先提问或标 `待确认`。
- 任何精确数字都要标注来源。

## Existing Codebase Rules

- 在提出变更前先探索当前结构，并遵循现有模式。
- 如果当前代码存在会影响本任务的边界问题，可以把有针对性的改进纳入方案。
- 不提出无关重构。
- 代码已经清楚表达的内容，不另造平行事实源。
- 如果任务过大，先拆成子系统或子项目，再决定每个子项目是否需要独立详细设计和计划。

## Design Presentation Rules

展示方案时要说明：

- 推荐方案是什么，为什么推荐。
- 备选方案各自适用什么条件。
- 哪些核心认知支撑了方案。
- 哪些 `待确认` 会影响最终设计或计划。
- 是否需要配套文件：Foundation Gate、类图、时序图、状态图、存储、运行时状态、API/事件契约、迁移兼容说明。

不要在 brainstorming 阶段把实现步骤铺开。完整 DDL、全量接口字段和完整状态表默认属于 `writing-detailed-design`；但 Foundation Gate 是例外，必须在进入详细设计前以模块化清单和配套文件方式完成地基确认。

## Document Handoff

当用户批准设计后，交接给 `writing-detailed-design` 时必须带上：

- 核心认知 `Vfinal`。
- 被批准的推荐方案和被拒绝的备选方案摘要。
- 已确认的 `DESIGN_ARTIFACTS_DIR/foundation-gate.md`。
- 用户明确确认的范围、限制和成功标准。
- 仍然存在但不阻塞详细设计的 `待确认`。
- 代码/文档/需求来源锚点。

如果核心认知或详细设计变化可能影响既有实现计划，必须在进入实现前同步计划，并记录到 `PLAN_CHANGELOG`。

## 视觉伴侣

视觉伴侣是一个浏览器工具，用于展示 mockup、图表和视觉化选项。它不是默认流程，只在用户看到视觉内容会更容易理解时使用。

如果预计接下来会出现视觉内容，必须单独发送这一条邀请，不能和其他问题混在一起：

> "Some of what we're working on might be easier to explain if I can show it to you in a web browser. I can put together mockups, diagrams, comparisons, and other visuals as we go. This feature is still new and can be token-intensive. Want to try it? (Requires opening a local URL)"

用户同意后，先阅读：

`skills/brainstorming/visual-companion.md`

判断标准只有一个：用户看见这个内容，会不会比只读文字更容易理解。

## Common Mistakes

- 跳过核心认知，直接给方案。
- 因为任务“看起来简单”就省略设计批准。
- 一次问多个澄清问题。
- 把详细设计正文或实现计划写进 brainstorming。
- 方案只描述推荐项，不讲备选和取舍。
- 代码证据出现后，不更新核心认知。
- 用户批准前开始实现。

## Completion Check

结束 brainstorming 前确认：

- 核心认知已经到 `Vfinal`，关键 `待确认` 已收敛或明确不阻塞。
- 用户已经批准推荐方案。
- Foundation Gate 已按功能模块列出 API、时序图、DDL/存储和一致性方案，且状态为 `confirmed`。
- 方案的范围适合一份详细设计；若不适合，已拆成子项目。
- 需要的配套设计产物已识别。
- 下一步是 `writing-detailed-design`，不是实现。
