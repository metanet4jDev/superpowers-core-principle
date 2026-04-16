---
name: writing-detailed-design
description: Use after brainstorming approval to write or update detailed design artifacts that reference an independent core-cognition document.
---

# 编写详细设计

## Purpose

把已批准的方案落成可评审、可计划、可实现的详细设计产物。详细设计引用独立核心认知文档，并为 `writing-plans` 提供事实源。

硬规则：

- 主文档讲决策、边界、方案比选和验证口径。
- 核心认知固定在 `CORE_COGNITION_PATH`，只引用，不复制展开。
- 配套文件承接图、接口、存储、运行时状态、迁移等实现细节。
- 同一事实只定义一次，其他位置只引用。
- 本技能不执行设计检查；整体检查由 `reviewing-design-artifacts` 在用户明确要求后触发。

## When to Use

Use when:

- `brainstorming` 已批准推荐方案，需要写入 `design-codex.md`。
- 设计涉及架构、接口、状态、存储、缓存、消息、迁移中的两类及以上。
- 主文档需要拆出类图、时序图、状态图、存储、API 或迁移说明。
- 设计变更后需要判断是否影响既有计划。

Skip when:

- 还在澄清需求或比较方案；先用 `brainstorming`。
- 只是从已批准设计生成实现步骤；用 `writing-plans`。
- 只是纯文案润色，不涉及结构、实体、状态、规则和边界。

## Required Inputs and Paths

开始前必须有：已批准设计方向、`core-principle` 的 `Vfinal`、来源锚点、范围/约束/成功标准、剩余 `待确认` 及其阻塞性。

写入或引用任务级路径前，先用 `superpowers:document-workspace-layout` 解析 `DESIGN_DOC_PATH`、`CORE_COGNITION_PATH`、`SPECS_DIR`、`DESIGN_ARTIFACTS_DIR`、`PLAN_PATH`、`PLAN_CHANGELOG`、`PLAN_WORKSPACE_DIR`、`TASK_WORKSPACE_DIR`。

## Output Bundle

默认输出 1 份主设计文档；核心认知是前置独立事实源，主文档只引用。配套文件按需要选择，不凑模板，默认写入 `DESIGN_ARTIFACTS_DIR`，主文档用相对路径引用。

| 文件 | 建议命名 | 作用 |
| --- | --- | --- |
| 核心认知文档 | `core-cognition.md` | 前置事实源；缺失时先回 `core-principle` 创建或更新 |
| 主设计文档 | `design-codex.md` | 背景、核心认知引用、方案比选、总体设计、功能点、验证、风险 |
| 类图/接口图 | `artifacts/<topic>-class-diagram.puml` | 核心接口、类、方法、依赖 |
| 时序图/状态图 | `artifacts/<topic>-sequence-or-state.puml` | 主链路、分支、状态迁移、补偿 |
| 存储变更 | `artifacts/<topic>-ddl.sql` / `artifacts/<topic>-storage.md` | 表、字段、索引、约束或非关系型结构 |
| 运行时状态 | `artifacts/<topic>-runtime-state.md` | cache、消息、任务、TTL、幂等、清理 |
| API/事件契约 | `artifacts/<topic>-openapi.yaml` / `artifacts/<topic>-contract.md` | Controller 契约、事件字段、错误语义、兼容 |
| 迁移兼容 | `artifacts/<topic>-migration.md` | 灰度、回滚、兼容窗口、数据迁移 |
| 联调前置条件 | `artifacts/<topic>-integration-prerequisites.md` | 环境、配置、外部依赖、调试顺序 |

## Main Document Contract

推荐结构：`背景和目标`、`范围和非目标`、`执行/上线/兼容约束`、`核心认知引用`、`方案比选`、`总体设计`、`功能点详细设计`、`配套文件`、`接口与验收`、`风险/待确认/受影响资产`。

规则：

- 正文只保留“为什么这样设计、影响什么、如何验证”。
- `核心认知引用` 只写 `CORE_COGNITION_PATH` 的相对路径、版本和本设计依赖的核心结论索引。
- 完整 DDL、全量接口字段、完整状态表、完整消息体放到配套文件。
- 方案比选必须区分推荐和备选；备选说明适用条件、复杂度、行为差异、兼容影响。
- 设计只服务特定范围时，必须写明适用边界。
- 涉及已上线代码、历史接口或联调依赖时，写清兼容约束、联调前置条件和受影响资产。

## Core Cognition Placement

核心认知始终是独立文档：

- `CORE_COGNITION_PATH` 不存在时，先用 `core-principle` 创建或更新到 `Vfinal`。
- `design-codex.md` 必须引用 `CORE_COGNITION_PATH`，不得复制核心认知全文。
- 详细设计新增事实会改变结构、持久化实体、关系、属性、状态或全局约束时，先更新核心认知，再更新详细设计。
- 功能点详细规则、执行路径、时序、图示、接口字段和验收条件只写在详细设计或配套文件。

## Function Point Contract

每个功能点至少覆盖：

- `功能`：动作、目的、作用对象、执行路径（主路径/失败路径/终止条件/序列图引用）。
- `参与实体`：实体、关系、属性、状态。
- `界限`：`given`、`when`、`then`、`exception`、`verify`。

要求：

- `given`：输入契约、实体关系、关键属性、初始状态。
- `when`：按场景列输入、关系、属性值、状态值、决策结果。
- `then`：按同一编号列输出、关系变化、属性变化、状态变化。
- `verify`：与 `then` 场景编号一一对应，说明验证方法、结果和有效性。

## Companion Artifact Rules

- 类图/接口图：只画本次新增或修改的核心接口、类、方法和依赖方向。
- 时序图：单条关键流程跨 `>=3` 个子系统或 `>=3` 个模块时必须补。
- 状态图：状态存在多分支、回退、重试、并行汇合、拒绝或超时时必须补。
- 存储变更：只写本次变更；关系型存储写主键、唯一键、索引、状态字段、时间字段。
- 运行时状态：写名称模板、字段、生命周期、写入方、读取方、清理策略、并发语义。
- API/事件契约：controller 接口统一按 OpenAPI/Apifox Controller 契约口径编写。可导入 Apifox 时优先输出 OpenAPI 3.0.3 YAML；只描述 controller 契约（路径、方法、参数、请求体、响应体、字段类型、字段可见性、nullable、返回分支、所属模块、兼容例外），不要混入 service、DAO、SQL、消息队列或内部调用链。非 controller 事件契约写入独立 `<topic>-contract.md`。历史协议只写本次增量。
- 迁移兼容：影响存量数据、历史调用方、灰度或回滚时必须补。
- 联调前置条件：外部配置、域名、回调、模板、账号、数据准备和推荐调试顺序可拆出独立文件。

用户禁止画图时，用等价文字说明并标注“图示待补”。

## User-Triggered Validation

设计检查不自动执行。写完或更新详细设计后，只能提示：

> "Detailed design saved. Tell me if you want me to run a design artifact check."

用户明确只要求验证详细设计时，使用 `validation-checklist.md`；只检查详细设计和配套文件，不检查核心认知完整性。

用户明确要求整体检查核心认知和详细设计时，使用 `reviewing-design-artifacts`。

## Traceability and Plan Sync

提交详细设计前检查：

- 主文档引用了必要配套文件。
- 主文档和配套文件没有大段重复。
- 功能点里的实体、关系、属性、状态能在定义章节定位。
- 关键属性至少在一个 `when` 或 `then` 中承担判断或变化作用。
- 状态图、正文、契约、存储说明不冲突。
- `then` 与 `verify` 场景编号一致。
- `TODO`、`TBD`、`待补`、`待确认` 不影响当前闭环。

详细设计变更后必须检查计划联动：

- `PLAN_PATH` 不存在：记录“不涉及既有计划同步”。
- `PLAN_PATH` 存在：检查是否影响任务、文件范围、测试或顺序。
- 影响计划时，调用 `writing-plans` 更新计划，并追加 `PLAN_CHANGELOG`。

## Completion Check

- 设计文档可作为 `writing-plans` 的事实来源。
- 核心认知、功能点、配套文件没有重复定义。
- 职责边界清楚：主文档讲决策，配套文件讲细节，计划讲执行。
