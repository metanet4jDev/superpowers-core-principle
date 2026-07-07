---
name: writing-detailed-design
description: Use after brainstorming approval to write or update detailed design artifacts that reference an independent core-cognition document.
---

# 编写详细设计

## 目标、边界与前置

把已批准的方案落成可评审、可计划、可实现的详细设计产物。详细设计引用独立核心认知文档，并为 `writing-plans` 提供事实源。
Use when：方案已获 `brainstorming` 批准，需要写入 `design-codex.md`；设计涉及架构、接口、状态、存储、缓存、消息、迁移中的两类及以上；需要拆出类图、时序图、状态图、存储、API 或迁移说明；或设计变更后需判断计划影响。
Skip when：仍在澄清或比选方案时用 `brainstorming`；只生成实现步骤时用 `writing-plans`；纯文案润色且不涉及结构、实体、状态、规则和边界时不使用本技能。
硬规则：

- 主文档讲决策、边界、方案比选和验证口径。
- 核心认知固定在 `CORE_COGNITION_PATH`，只引用，不复制展开。
- 详细设计前必须已有 `DESIGN_ARTIFACTS_DIR/foundation-gate.md`，且状态为 `confirmed`。
- 配套文件承接图、接口、存储、运行时状态、迁移等实现细节。
- 同一事实只定义一次，其他位置只引用。
- 本技能不执行设计检查；整体检查由 `reviewing-design-artifacts` 在用户明确要求后触发。

开始前必须有：已批准设计方向、`core-principle` 的 `Vfinal`、已确认的 `foundation-gate.md`、真实代码库/数据库/依赖环境证据、来源锚点、范围/约束/成功标准、剩余 `待确认` 及其阻塞性。

写入或引用任务级路径前，先用 `superpowers:document-workspace-layout` 解析 `DESIGN_DOC_PATH`、`CORE_COGNITION_PATH`、`SPECS_DIR`、`DESIGN_ARTIFACTS_DIR`、`PLAN_PATH`、`PLAN_CHANGELOG`、`PLAN_WORKSPACE_DIR`、`TASK_WORKSPACE_DIR`。

`DESIGN_ARTIFACTS_DIR/foundation-gate.md` 的总门禁状态不是 `confirmed` 时，停止写详细设计，先回到 `brainstorming` 完成地基确认。

## Output Bundle

默认输出 1 份主设计文档；核心认知是前置独立事实源，主文档只引用。配套文件按需要选择，不凑模板，默认写入 `DESIGN_ARTIFACTS_DIR`，主文档用相对路径引用。

| 文件 | 建议命名 | 作用 |
| --- | --- | --- |
| 核心认知文档 | `core-cognition.md` | 前置事实源；缺失时先回 `core-principle` 创建或更新 |
| 地基确认门禁 | 逻辑路径：`DESIGN_ARTIFACTS_DIR/foundation-gate.md`；主文档相对引用：`artifacts/foundation-gate.md` | 按功能模块索引 API、时序图、DDL/存储和一致性方案；必须先人工确认 |
| 主设计文档 | `design-codex.md` | 背景、核心认知引用、方案比选、总体设计、功能点、验证、风险 |
| 类图/接口图 | `artifacts/<topic>-class-diagram.puml` | 只画本次新增或修改的核心接口、类、方法和依赖方向 |
| 时序图/状态图 | `artifacts/<module>/<module>-sequence.puml` / `artifacts/<topic>-state.puml` | 每模块至少一张主时序，覆盖主链路、分支、状态迁移、补偿；复杂流程再拆。状态有分支、回退、重试、并行汇合、拒绝或超时时补状态图 |
| 存储变更 | `artifacts/<module>/<module>-ddl.sql` / `artifacts/<module>/<module>-storage.md` | 只写本次变更；写清表、字段、索引、约束或非关系型结构；关系型存储明确主键、唯一键、状态和时间字段 |
| 运行时状态 | `artifacts/<topic>-runtime-state.md` | cache、消息、任务的名称模板、字段、生命周期、写入方、读取方、TTL、幂等、清理策略、并发语义 |
| API/事件契约 | `artifacts/<module>/<module>-openapi.yaml` / `artifacts/<topic>-contract.md` | Controller 按 OpenAPI/Apifox 口径；可导入时优先 OpenAPI 3.0.3 YAML，只含路径、方法、参数、请求体、响应体、字段类型/可见性/nullable、返回分支、所属模块、错误语义、兼容例外，不混入 service、DAO、SQL、消息队列或内部调用链。非 Controller 事件字段和错误语义写独立契约；历史协议只写增量 |
| 数据一致性 | `artifacts/<module>/<module>-consistency.md` | 内部、跨子系统、第三方都写强一致/最终一致选择、事务边界、幂等键、唯一约束、重试、补偿、对账、失败终态和人工确认 |
| 迁移兼容 | `artifacts/<topic>-migration.md` | 影响存量数据、历史调用方、灰度或回滚时写灰度、回滚、兼容窗口、数据迁移 |
| 联调前置条件 | `artifacts/<topic>-integration-prerequisites.md` | 环境、配置、外部依赖、域名、回调、模板、账号、数据准备、推荐调试顺序 |

用户禁止画图时，用等价文字说明并标注“图示待补”。

## Foundation Gate Contract

`foundation-gate.md` 是详细设计前的门禁索引，固定写入 `DESIGN_ARTIFACTS_DIR/foundation-gate.md`。主文档从 `SPECS_DIR` 相对引用 `artifacts/foundation-gate.md`。不要在 `DESIGN_ARTIFACTS_DIR` 内再创建嵌套的 `artifacts/foundation-gate.md`。

本文件不复制完整 DDL、OpenAPI 或时序图。完整内容放到模块目录下的配套文件，并由本文件引用。

文件必须使用以下结构：

```markdown
# Foundation Gate

## 1. 总门禁状态
- 任务：
- 核心认知：
- 状态：draft / pending-human-confirmation / confirmed / blocked
- 阻塞模块：
- 人工确认人：
- 确认时间：

## 2. 功能模块总览
| 模块 | 功能点 | API | 时序图 | DDL/存储 | 一致性方案 | 状态 | 阻塞 |
| --- | --- | --- | --- | --- | --- | --- | --- |

## 3. 模块：<模块名>

### 3.1 模块边界
- 代码库证据：
- 数据库证据：
- 依赖环境证据：
- 模块职责：
- 不负责：
- 事实源：
- 依赖模块：

### 3.2 功能点清单
| 功能点 | 业务目标 | 发起端 | 主时序图 | 关联 API | 关联表 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |

### 3.3 API 清单（按调用端/tag）
| 调用端/tag | 功能点 | Method | Path | 请求 VO | 响应 VO | 副作用 | 幂等 | 状态 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 3.4 时序图清单
| 场景 | 发起方 | 文件 | 覆盖 API | 关联表 | 可走通性 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |

### 3.5 DDL / 存储约束
| 表 | 变更类型 | 软删 | 唯一约束策略 | 索引依据 | DDL 文件 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |

### 3.6 数据一致性方案
| 场景 | 范围 | 强一致/最终一致 | 事务边界 | 幂等/补偿 | 对账/重试 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |

### 3.7 可走通性检查
- 前置输入是否已存在：
- 是否依赖后置步骤结果：
- API、时序图、DDL 是否对齐：
- 阻塞项：

### 3.8 人工确认
- 结论：confirmed / blocked
- 意见：
```

要求：

- 功能模块是一级组织维度；不要按 Controller、表或接口方法拆主章节。
- 主事实归属哪个模块，就放哪个模块；跨模块处只引用。
- API、主时序图和 DDL 必须基于真实代码库、真实数据库环境和真实依赖环境。无法取得真实证据时，对应模块状态必须是 `blocked` 或 `pending-human-confirmation`，不能写成 `confirmed`。
- 代码库证据至少包括相关 Controller/路由/VO/Service/调用方或配置文件；数据库证据至少包括真实库表结构、索引、数据量或存量数据校验；依赖环境证据至少包括网关/BFF、Nacos/配置、MQ topic、Redis key、第三方平台、回调或鉴权入口中的相关项。
- 总门禁状态为 `confirmed` 前，不得写 `design-codex.md`。
- 任一模块存在阻塞项时，总门禁状态必须是 `blocked` 或 `pending-human-confirmation`。

## Main Document and Core Cognition Contract

推荐结构：`背景和目标`、`范围和非目标`、`执行/上线/兼容约束`、`核心认知引用`、`方案比选`、`总体设计`、`功能点详细设计`、`配套文件`、`接口与验收`、`风险/待确认/受影响资产`。

规则：

- 正文只保留“为什么这样设计、影响什么、如何验证”。
- `核心认知引用` 只写 `CORE_COGNITION_PATH` 的相对路径、版本和本设计依赖的核心结论索引。
- `CORE_COGNITION_PATH` 不存在时，先用 `core-principle` 创建或更新到 `Vfinal`；主文档不得复制核心认知全文。
- 新事实改变结构、持久化实体、关系、属性、状态或全局约束时，先更新核心认知，再更新详细设计。
- `配套文件` 必须引用 `foundation-gate.md`，并说明本设计消费的模块清单。
- 完整 DDL、全量接口字段、完整状态表、完整消息体放到配套文件。
- 功能点详细规则、执行路径、时序、图示、接口字段和验收条件只写在详细设计或配套文件。
- 方案比选必须区分推荐和备选；备选说明适用条件、复杂度、行为差异、兼容影响。
- 设计只服务特定范围时，必须写明适用边界。
- 涉及已上线代码、历史接口或联调依赖时，写清兼容约束、联调前置条件和受影响资产。

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

## User-Triggered Validation

设计检查不自动执行。写完或更新详细设计后，只能提示：

> "Detailed design saved. Tell me if you want me to run a design artifact check."

用户明确只要求验证详细设计时，使用 `validation-checklist.md`；只检查详细设计和配套文件，不检查核心认知完整性。

用户明确要求整体检查核心认知和详细设计时，使用 `reviewing-design-artifacts`。

## Traceability, Plan Sync, and Completion

提交详细设计前检查：

- 主文档引用了必要配套文件。
- `foundation-gate.md` 已确认，且 API、时序图、DDL/存储和数据一致性方案与主文档一致。
- API、主时序图和 DDL 的真实代码库、真实数据库环境和真实依赖环境证据已记录。
- 主文档和配套文件没有大段重复。
- 功能点里的实体、关系、属性、状态能在定义章节定位。
- 关键属性至少在一个 `when` 或 `then` 中承担判断或变化作用。
- 状态图、正文、契约、存储说明不冲突。
- `then` 与 `verify` 场景编号一致。
- `TODO`、`TBD`、`待补`、`待确认` 不影响当前闭环。
- 设计文档可作为 `writing-plans` 的事实来源。
- 核心认知、功能点、配套文件没有重复定义。
- 职责边界清楚：主文档讲决策，配套文件讲细节，计划讲执行。

详细设计变更后必须检查计划联动：

- `PLAN_PATH` 不存在：记录“不涉及既有计划同步”。
- `PLAN_PATH` 存在：检查是否影响任务、文件范围、测试或顺序。
- 影响计划时，调用 `writing-plans` 更新计划，并追加 `PLAN_CHANGELOG`。
