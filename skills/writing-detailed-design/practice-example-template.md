# 详细设计实践模板（历史 spec 抽象版）

本模板用于直接编写 `specs/design-codex.md` 和配套 `specs/artifacts/*`。它不是模板说明书，而是历史设计实践的可复制写法汇总。

使用规则：

1. 先完成 `core-cognition.md` 到 `Vfinal`，再写 `artifacts/foundation-gate.md`，最后写 `design-codex.md`。
2. 主文档只写决策、边界、方案比选、功能点和验证口径；实体属性、状态集合、关系和全局约束引用 `core-cognition.md`。
3. API 全量字段、DDL、时序图、事件契约、联调前置条件放到 `artifacts/`，主文档只引用。
4. 示例均来自历史 `harness-hh/spec`，只压缩表达，不虚构接口、表名、状态、topic 或字段。

## 0. 输出包模板

```text
specs/
  core-cognition.md
  design-codex.md
  artifacts/
    foundation-gate.md
    <module>-sequence.puml
    <module>-openapi.yaml
    <module>-ddl.sql
    <module>-contract.md
    <topic>-integration-prerequisites.md
plans/
  plan-changelog.md
  main/
    implementation-plan.md
```

实践示例：

```md
- 主文档：`specs/design-codex.md`
- `CORE_COGNITION_PATH`：[`./core-cognition.md`](./core-cognition.md)
- `DESIGN_ARTIFACTS_DIR`：[`./artifacts`](./artifacts)
- `FOUNDATION_GATE`：[`./artifacts/foundation-gate.md`](./artifacts/foundation-gate.md)，状态 `confirmed-with-risk`
- `PLAN_PATH`：[`../plans/main/implementation-plan.md`](../plans/main/implementation-plan.md)
- `PLAN_CHANGELOG`：[`../plans/plan-changelog.md`](../plans/plan-changelog.md)
```

来源：`harness-hh/spec/v1-5-0-seven-hospitals/specs/design-codex.md`

## 1. 主文档头部

```md
# <业务主题>详细设计

- 日期：<YYYY-MM-DD>
- 任务：`<task-name>`
- 适用范围：`<repo-a>`、`<repo-b>`
- 文档口径：本文保留设计决策、边界、方案和验证口径；稳定事实以 [`core-cognition.md`](./core-cognition.md) 为主落点。

## 配套文件

1. 核心认知：[`core-cognition.md`](./core-cognition.md)
2. 地基门禁：[`artifacts/foundation-gate.md`](./artifacts/foundation-gate.md)
3. 主时序图：[`artifacts/<module>-sequence.puml`](./artifacts/<module>-sequence.puml)
4. API 契约：[`artifacts/<module>-openapi.yaml`](./artifacts/<module>-openapi.yaml)
5. DDL / 存储：[`artifacts/<module>-ddl.sql`](./artifacts/<module>-ddl.sql)
6. 联调前置条件：[`artifacts/<topic>-integration-prerequisites.md`](./artifacts/<topic>-integration-prerequisites.md)
```

跨系统设计实践：

```md
- 适用范围：`yt-plat`、`yt-pha-opm-server`、`udbnew`、`pbs-token`、`pbs-message`
- 文档口径：本文保留设计决策、边界、方案和验证口径；稳定事实以核心认知为主落点，接口、存储、联调和 PBS 格式由配套文件承接。
```

轻量单接口设计实践：

```md
- 适用范围：`pbs-token`
- 文档口径：本文保留设计决策、接口设计和验证口径；稳定事实以核心认知为主落点。
```

来源：

- `harness-hh/spec/v1-38-baseline-iteration/specs/design-codex.md`
- `harness-hh/spec/wechat-menu-delete/specs/design-codex.md`

## 2. 背景和目标

模板：

```md
## 背景和目标

<用 1 到 3 段说明当前问题，必须写清“现状为什么不够”和“本次要修正什么”。>

本设计目标：

1. <目标 1>
2. <目标 2>
3. <目标 3>

成功标准：

1. <可验证标准 1>
2. <可验证标准 2>
3. <可验证标准 3>
```

生产一致性问题示例：

```md
医生在 `/d/90120/930` 作废处方后，平台处方已经 `ABOLITION=1`，但医保上传链路仍在执行，后续 `olt_prescription_med_insure.rx_stas_codg='1'`，医保侧处方生效并发生结算。现有每小时补偿作废过慢，等到补偿时医保已返回“处方已使用”。

本次目标：

1. 医生作废前先由医保助手尝试医保 7104 作废，明确区分“可继续平台作废”和“医保已使用不可作废”。
2. 平台处方作废后，通过 RocketMQ 延时消息做医保作废补偿，避免作废和上传并发窗口导致医保侧残留成功处方。
3. 作废后终止处方上传任务，防止后续自动上传继续处理同一处方。
4. 不修改医保上传主流程相关代码。
```

接口兼容问题示例：

```md
PRD 要求 `/auth/system/pageUserNoAuth` 能支持查询冻结用户，用于报销数据导入相关场景识别弃用用户。经代码和数据库核实，报销导入本身已经允许冻结用户作为报销人匹配，组织结构管理也允许保存或编辑冻结用户。

成功标准：

1. `status` 为空且不传新增字段时，仍只返回 `enabled`。
2. 传 `includeStatuses=["enabled","frozen"]` 时，一次返回正常和冻结用户。
3. `disabled` 用户不作为可选用户返回。
4. 报销导入和组织结构管理冻结用户能力保持现状。
```

来源：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/报销数据导入增加弃用用户校验/design-codex.md`

## 3. 范围和非目标

模板：

```md
## 范围和非目标

范围：

1. <接口 / 模块 / 链路 / 表 / 配置 / 消息>
2. <接口 / 模块 / 链路 / 表 / 配置 / 消息>

非目标：

1. 不<改变历史接口 / 新增表 / 改上传主流程 / 改编辑链路>
2. 不<处理历史脏数据 / 做前端页面设计 / 替换旧调用方>
3. 不<把 service、DAO、SQL、消息链路写进 OpenAPI>
```

新增接口但不改历史接口示例：

```md
范围：

1. 在 `yt-med-pat-server` 新增 `POST /r/{srcId}/50004/141`。
2. 在 `yt-med-reg-server` 新增 `POST /r/{hosId}/60070/151`。
3. 在 `yt-med-cfg-server` 新增 `POST /r/{srcId}/50001/312`。
4. 新增机构参数 `integratedPatRecordSwitch` 和 `regDeadlineTimeLimit`。

非目标：

1. 不修改历史接口路径、请求字段或返回语义：`50004/140`、`50004/300`、`50004/150`、`60070/150`。
2. 不新增数据库表、DDL、索引或迁移脚本。
3. 不把 service、DAO、SQL、消息或内部调用链写进 OpenAPI 契约。
```

旧逻辑保留、新能力隔离示例：

```md
本文不覆盖：

1. 历史联查 SQL 下线。
2. 跨模块统一替换所有旧调用点。
3. 新库表、新缓存、新离线回填任务。
4. 实现代码与测试用例。
```

来源：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/design-codex.md`
- `harness-hh/spec/online-consultation-drug-search-history-prescription-validation/specs/design-codex.md`

## 4. 执行、上线和兼容约束

模板：

```md
## 执行、上线和兼容约束

1. 已上线接口兼容：<历史调用不传新增字段时行为不变>。
2. 默认值兼容：<配置缺失 / 字段缺失 / 未传开关时的默认口径>。
3. 参数优先级：<新旧字段同时存在时谁优先生效>。
4. 上线顺序：<DDL -> 配置 -> 代码 / 代码 -> 配置>。
5. 回滚策略：<关闭开关 / 前端隐藏入口 / DDL 不反向回滚>。
6. 真实环境依赖：<Redis/Nacos/网关/BFF/HIS/MQ/topic/appId/token_isv>。
```

配置开关兼容示例：

```md
- 已上线接口兼容：历史调用不传新增 DTO 字段时，行为不变（默认视为开启）。
- 配置缺失兼容：开关值不存在时按默认开启处理，不阻止保存。
- 参数优先级：开关值由服务端读取并透传，不接受前端自行传值覆盖。
- 上线顺序：先执行 DDL 变更 → 创建配置项定义 → 部署业务代码。
- 回滚策略：删除配置项值即可恢复默认开启状态；DDL 不反向回滚。
```

状态集合兼容示例：

```md
- 已上线接口兼容：历史调用不传 `includeStatuses` 时行为不变。
- 查询结果兼容：`status` 为空继续只返回 `enabled`，避免默认下拉突然出现冻结用户。
- 参数优先级：`status` 与 `includeStatuses` 同时存在时，`status` 优先生效。
- 状态白名单：`includeStatuses` 只接受 `enabled`、`frozen`，不能通过该字段返回 `disabled`。
```

来源：

- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/标品库增加药品校验逻辑/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/报销数据导入增加弃用用户校验/design-codex.md`

## 5. 核心认知引用

模板：

```md
## 核心认知引用

- 路径：[`core-cognition.md`](./core-cognition.md)
- 版本：`Vfinal`
- 用法：持久化事实、关系、状态和全局约束以核心认知为唯一主落点；本文只引用，不复制展开。

本设计依赖的核心结论：

1. 结构：<事实源、编排层、外部依赖归属>。
2. 持久化数据模型：<本次新增或复用的稳定实体>。
3. 持久化关系：<一对一 / 一对多 / 唯一定位 / 依附关系>。
4. 持久化状态：<状态集合和禁止迁移>。
5. 功能：<本设计覆盖的功能能力>。
6. 全局约束：<跨模块不变量>。
```

跨系统核心结论示例：

```md
1. 结构：`yt-pha-opm-server` 是登录主编排层；微信官方 API 统一由 PBS 微信适配层承接；`udbnew` 持有 `openId -> usId` 绑定事实。
2. 持久化数据模型：新增登录编排态只落 `opm_wechat_qr_scene` 与 `opm_wechat_login_ticket`；`pat_user`、`pat_user_r_hos`、`udb_third_bind` 继续复用现网模型。
3. 持久化状态：`opm_wechat_login_ticket.status` 只保留 `PROCESSING / SUCCESS / INVALID`。
4. 全局约束：OPM 不保存 `appSecret`，不直连微信官方 API；本人档案与就诊卡未就绪前不签发正式业务 token。
```

配置开关核心结论示例：

```md
- `olt_drug_library_medinsure.medical_insurance_code` 当前为 longtext 且无索引，全表扫描；本期优化为 varchar(128) 并补建普通索引。
- 单个新增与批量导入共用 `batchSave(...)` 管道，必须一起改。
- 开关控制点只涉及 `DrugCatalogueDao.selectMedinsureLibraryDrug` 调用。
- 本地重复性校验、编码冲突校验、医保码格式校验不受开关影响。
```

来源：

- `harness-hh/spec/v1-38-baseline-iteration/specs/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/标品库增加药品校验逻辑/design-codex.md`

## 6. Foundation Gate

模板：

```md
# Foundation Gate

## 1. 总门禁状态

- 任务：`<task-name>`
- 核心认知：[`../core-cognition.md`](../core-cognition.md)，版本 `Vfinal`
- 状态：`draft / pending-human-confirmation / confirmed / blocked`
- 阻塞模块：<模块列表或无>
- 人工确认人：<name>
- 确认时间：<YYYY-MM-DD>

## 2. 功能模块总览

| 模块 | 功能点 | API | 时序图 | DDL/存储 | 一致性方案 | 状态 | 阻塞 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| <模块> | <功能点> | `<METHOD /path>` | `<file>` | <DDL 或“无 DDL 变更”> | <强一致/最终一致/只读> | `<status>` | <阻塞项> |
```

当前技能标准状态只有 `draft / pending-human-confirmation / confirmed / blocked`。历史 spec 存在 `confirmed-with-risk` 实践；新文档若证据不足，优先写 `pending-human-confirmation`，并在保留风险中列证据缺口。只有团队明确接受“带风险确认”状态时，才使用 `confirmed-with-risk`。

confirmed 示例：

```md
- 任务：真实目录单个添加药品查询逻辑调整
- 核心认知：`../core-cognition.md`
- 状态：confirmed
- 阻塞模块：无
- 确认人：胡先生
- 确认时间：2026-05-15
```

confirmed-with-risk 示例：

```md
- 任务：`v1-5-0-seven-hospitals`
- 核心认知：[`../core-cognition.md`](../core-cognition.md)，版本 `Vfinal-confirmed-with-risk`
- 状态：`confirmed-with-risk`
- 保留风险模块：整合建档、挂号截止查询、配置中心参数读取
- 人工确认人：胡先生
- 确认时间：2026-05-08
```

模块总览示例：

```md
| 模块 | 功能点 | API | 时序图 | DDL/存储 | 一致性方案 | 状态 | 阻塞 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 标准库查询 | 国家医保药品代码精确查询第一条 | `POST /r/{yunId}/80200/dp/525` | `artifacts/real-drug-medins-end-to-end-sequence.puml` | 无 DDL 变更 | 只读查询，无事务 | confirmed | 无 |
| 真实目录保存 | 单个医保药品简化新增、完整字段新增兼容 | `POST /r/{yunId}/80200/dp/730` | `artifacts/real-drug-medins-single-save-sequence.puml` | 无 DDL 变更 | 单次保存沿用现有事务边界；兼容分支保持原链路 | confirmed | 无 |
```

来源：

- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/真实目录单个添加药品查询逻辑调整/artifacts/foundation-gate.md`
- `harness-hh/spec/v1-5-0-seven-hospitals/specs/artifacts/foundation-gate.md`

## 7. Foundation Gate 模块块

模板：

```md
## 3. 模块：<模块名>

### 3.1 模块边界

- 代码库证据：
  - `<Controller#method>`：<入口或缺口>。
  - `<Service#method>`：<复用或新增落点>。
- 数据库证据：
  - `<db.table.column>`：<字段、索引、数据量、状态或未验证项>。
- 依赖环境证据：
  - <Redis / Nacos / MQ / 网关 / 第三方 / 鉴权入口>。
- 模块职责：<本模块负责什么>。
- 不负责：<本模块明确不做什么>。
- 事实源：<表 / core-cognition 章节 / 外部系统>。
- 依赖模块：<依赖列表>。

### 3.2 功能点清单

| 功能点 | 业务目标 | 发起端 | 主时序图 | 关联 API | 关联表 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |

### 3.3 API 清单（按调用端/tag）

| 调用端/tag | 功能点 | Method | Path | 请求 VO | 响应 VO | 副作用 | 幂等 | 状态 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 3.5 DDL / 存储约束

| 表 | 变更类型 | 软删 | 唯一约束策略 | 索引依据 | DDL 文件 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |

### 3.6 数据一致性方案

| 场景 | 范围 | 强一致/最终一致 | 事务边界 | 幂等/补偿 | 对账/重试 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |
```

代码、库表、依赖证据示例：

```md
- 代码库证据：
  - `yt-pha-rp-server/.../RestDrugCatalogueServiceImpl.java:2551-2603（queryDpmMatchDrugInfo）`
  - `yt-pha-rp-server/.../DrugLibraryDao.xml:1038-1046（medical_insurance_code 精确条件）`
- 数据库证据：`olt_drug_library.medical_insurance_code` 为普通索引，不唯一；测试库同一医保码存在多条记录
- 依赖环境证据：测试库通过 `environment/db-config/mysql/test.env` 只读查询验证
- 模块职责：按国家医保药品代码精确查询标准库第一条，返回现有 `DrugCatalogueVo` 结构
- 不负责：写真实目录，回写标准库
- 事实源：`olt_drug_library`
```

无 DDL 但要写清存储约束示例：

```md
| 表 | 变更类型 | 软删 | 唯一约束策略 | 索引依据 | DDL 文件 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |
| `olt_drug_library` | 无变更 | 沿用现状 | `medical_insurance_code` 不唯一，按 `DRUG_LIBRARY_ID ASC` 稳定取第一条 | 已有普通索引 | 无 | confirmed |
```

来源：`harness-hh/spec/v1-39-0-version-iteration/specs/v2/真实目录单个添加药品查询逻辑调整/artifacts/foundation-gate.md`

## 8. 方案比选

模板：

```md
## 方案比选

| 方案 | 描述 | 优点 | 缺点 | 结论 |
| --- | --- | --- | --- | --- |
| 推荐方案 | <做法> | <为什么适合本任务> | <代价> | 采用 |
| 备选方案 A | <做法> | <优点> | <为什么不适合> | 不采用 |
| 备选方案 B | <做法> | <优点> | <为什么不适合> | 不采用 |

推荐方案：`<方案名>`。
```

最终一致性方案比选示例：

```md
| 方案 | 描述 | 优点 | 缺点 | 结论 |
| --- | --- | --- | --- | --- |
| A | 平台先作废，再完全依赖 MQ 补偿医保作废 | 对 RP 改动少 | 医保已使用时仍会本地作废，用户无法被即时拦截 | 不采用 |
| B | 医保助手先尝试医保作废；成功才调用 RP 作废；“处方已使用”拦截；其他失败走 RP 作废和 MQ 补偿 | 能拦截医保已使用，兼容上传并发窗口，符合最终一致性目标 | 需要 RP、PLAT、医保助手协同 | 采用 |
| C | 医保作废必须成功才允许平台作废，其他失败全部阻断 | 一致性最强 | 上传中场景会误阻断平台作废，无法解决并发窗口 | 不采用 |
```

可组合校验方案比选示例：

```md
### 方案 B：新增单表查询链路，Java 内存拼装，再接可组合校验链

- 优点：满足“单表查询”约束；旧逻辑不动；查询和校验职责分离；后续可分别扩展真实目录校验与虚拟目录校验。
- 缺点：调用链更长，需要补 DTO / VO / 校验接口和更多单元测试。
- 结论：推荐。
```

来源：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`
- `harness-hh/spec/online-consultation-drug-search-history-prescription-validation/specs/design-codex.md`

## 9. 总体设计

模板：

```md
## 总体设计

### 9.1 系统职责 / 模块职责

| 系统/模块 | 职责 |
| --- | --- |
| <系统 A> | <事实源、编排、执行或适配职责> |

### 9.2 API 全景

| 调用端/tag | 所属模块 | 功能点 | Method | Path | 请求 VO | 响应 VO | 副作用 | 幂等 | 兼容说明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 9.3 关键实现落点

| 能力 | 目标模块 | 目标入口 | 复用锚点 |
| --- | --- | --- | --- |

### 9.4 实现约束

1. <约束 1>
2. <约束 2>
```

跨系统职责示例：

```md
| 系统 | 职责 |
| --- | --- |
| 医生工作站 | 承接医生点击作废；根据医保助手结果决定是否继续调用 RP 作废 |
| 医保助手 | 获取 7104 参数，调用医保服务作废处方，识别医保返回结果 |
| yt-pha-rp-server | 执行 `/d/90120/930` 平台作废，成功后发送医保作废补偿 MQ |
| yt-plat-all | 管理医保处方记录、医保上传任务、医保作废任务；消费补偿 MQ |
| RocketMQ | 承接作废后延时补偿检查 |
```

API 全景示例：

```md
| 调用端/tag | 所属模块 | 功能点 | Method | Path | 请求 VO | 响应 VO | 副作用 | 幂等 | 兼容说明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| H5端 / `seven-hospitals-pat` | 整合建档 | PAT 整合新增就诊人与建档 | POST | `/r/{srcId}/50004/141` | `IntegratedPatRecordRequest` | `ResultVo<IntegratedPatRecordResult>`；开关关闭时 `data=null` | 开关关闭无业务副作用；成功写就诊人和院内档案 | 未新增幂等键，复用历史校验和表约束 | 新增接口，不替换 `140/300/150` |
```

查询链路步骤示例：

```md
`/80200/dp/525` 的入参携带 `medicalInsuranceCode` 或 `medInsCode` 时进入医保码精确查询模式：

1. OPM Controller 继续接收 `DrugCatalogueDto`。
2. RP Service 归一医保码字段：优先 `medicalInsuranceCode`，为空则取 `medInsCode`。
3. 标准库 DAO 执行 `medical_insurance_code = ? order by DRUG_LIBRARY_ID asc limit 1`。
4. 命中时转换为现有 `DrugCatalogueVo`，返回列表最多 1 条。
5. 未命中时返回空列表。
```

来源：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`
- `harness-hh/spec/v1-5-0-seven-hospitals/specs/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/真实目录单个添加药品查询逻辑调整/design-codex.md`

## 10. 事件契约 / 消息契约

模板：

````md
### <事件名>事件契约

| 字段 | 值 |
| --- | --- |
| topic | `<topic>` |
| tag | `<tag>` |
| producer | `<producer>` |
| consumer | `<consumer>` |
| delayTimeLevel | `<level>` |
| maxRetryTimes | `<n>` |

消息体：

```json
{
  "<key>": "<value>"
}
```

处理要求：

1. <幂等键>。
2. <重试和终止规则>。
3. <哪些字段不复用历史任务字段>。
````

医保作废补偿示例：

````md
| 字段 | 值 |
| --- | --- |
| topic | `topic_drug_medical_insurance_pres` |
| tag | `PRES_CANCEL_CONSISTENCY` |
| producer | `yt-pha-rp-server` |
| consumer | `yt-plat-all` |
| delayTimeLevel | `4`，即 30 秒 |
| maxRetryTimes | `3` |

消息体：

```json
{
  "pscriptId": 1833041,
  "hosId": 107263,
  "retryTimes": 0,
  "source": "PRESCRIPTION_ABOLISH"
}
```

处理要求：

1. 消费端按 `pscriptId + hosId` 幂等处理。
2. 重试消息只递增 `retryTimes`。
3. 达到最大重试后不再发送新消息，保留日志供排查。
4. 这里的 `retryTimes` 是 MQ 补偿检查轮次，不写入 `med_prescription_upload_task.retry_times`。
````

来源：`harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`

## 11. 存储设计和 DDL

模板：

```md
### 存储设计

| 表 | 用途 | 本次变更 |
| --- | --- | --- |
| `<table>` | <用途> | <新增 / 修改 / 复用> |

### DDL 文件

- 路径：[`artifacts/<module>-ddl.sql`](./artifacts/<module>-ddl.sql)
- 上线顺序：<DDL -> 配置 -> 代码>
- 回滚策略：<是否反向回滚>
- 验证方法：<SHOW COLUMNS / SHOW INDEX / EXPLAIN>
```

复用存量表示例：

```md
| 表 | 用途 |
| --- | --- |
| `olt_prescription_ol` | 平台处方主状态，读取 `ABOLITION` 判断是否已作废 |
| `olt_prescription_med_insure` | 医保处方状态，读取和更新 `rx_stas_codg`、`invoken_result` |
| `med_prescription_upload_task` | type=1 上传任务、type=2 作废任务 |
```

DDL 文件示例：

```sql
ALTER TABLE med2.olt_drug_library_medinsure
    MODIFY COLUMN medical_insurance_code VARCHAR(128) NOT NULL DEFAULT ''
    COMMENT '国家医保药品代码';

CREATE INDEX idx_medical_insurance_code
    ON med2.olt_drug_library_medinsure (medical_insurance_code);
```

验证方法示例：

```sql
SHOW COLUMNS FROM med2.olt_drug_library_medinsure LIKE 'medical_insurance_code';
SHOW INDEX FROM med2.olt_drug_library_medinsure WHERE Key_name = 'idx_medical_insurance_code';
EXPLAIN SELECT COUNT(*) FROM med2.olt_drug_library_medinsure WHERE medical_insurance_code = 'TEST';
```

来源：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/标品库增加药品校验逻辑/artifacts/ddl.sql`

## 12. 功能点详细设计

模板：

```md
### 功能点 <n>：<功能点名称>

功能：<动作 + 目的 + 作用对象>。

参与实体：

- `<EntityA>`
- `<EntityB>`

执行路径：

1. <入口>
2. <关键编排>
3. <状态 / 关系 / 属性变化>
4. <返回>

序列图：[`artifacts/<module>-sequence.puml`](./artifacts/<module>-sequence.puml)

| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| F1 | <前置条件、输入契约、实体关系、初始状态> | <触发动作和判断条件> | <输出、关系变化、属性变化、状态变化> | <异常口径> | <验证方法和预期结果> |
```

查询功能点示例：

```md
### 功能点 1：医保码精确查询标准库第一条

功能：输入国家医保药品代码，后端精确查询标准库第一条，供页面展示默认参数和标准参数。

参与实体：`DrugCatalogueDto`（接收 `dpId`、医保码、`type`）、`DrugLibraryVo`（承载标准库查询结果）、`olt_drug_library`（事实源）

| 编号 | when | then | exception | verify |
| --- | --- | --- | --- | --- |
| Q1 | 医保码命中一条或多条标准库记录 | 返回列表最多 1 条，取 `DRUG_LIBRARY_ID` 最小记录 | 无 | 用 `T001700368` 多次查询，返回同一 `drug_library_code` |
| Q2 | 医保码未命中 | 返回空列表 | 无 | 用不存在医保码查询，不报错且不写库 |
| Q3 | `dpId` 为空 | 沿用现有“入参_药商主键不能为空” | 请求失败 | 调用接口确认返回现有错误语义 |
```

历史处方带入示例：

```md
| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| N3 | 报价请求带 `historyPscriptId`、`historyPMedDetailId`，且历史处方 `HOS_ID != 当前 hosId` | 调用 `/getDrugQuote` | 频次和用法都置空；不再继续做历史映射和目录默认值回退 | 无 | 核对返回字段为空，且不会命中目录默认值 |
| N4 | 历史处方属于当前机构，历史频次/用法都能映射当前机构字典 | 调用 `/getDrugQuote` | 优先返回历史映射值；`frequencyQty`、`presSustainedDays` 继续沿历史值透传 | 无 | 核对返回编码和名称来自当前机构字典，而不是原始历史字符串 |
| N5 | 历史处方属于当前机构，但历史频次或历史用法无法映射当前字典 | 调用 `/getDrugQuote` | 对应字段回退到当前命中目录默认值；仍按字段独立判断 | 目录无默认值或默认值不可用时，对应字段置空 | 分别构造“频次失败、用法成功”和“用法失败、频次成功”两套用例 |
```

状态集合兼容示例：

```md
| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| S1 | 请求体 `status=""`，未传 `includeStatuses` | 查询 `pageUserNoAuth` | 只返回 `enabled` 用户 | 无 | 接口响应中无 `frozen/disabled` |
| S2 | 请求体 `status=""`，`includeStatuses=["enabled","frozen"]` | 查询 `pageUserNoAuth` | 返回 `enabled` 和 `frozen` 用户 | 无 | test.env 中 `yt_plat_hospital` 存在两类用户，可查到两类状态 |
| S3 | 请求体 `status="frozen"`，同时传 `includeStatuses=["enabled","frozen"]` | 查询 `pageUserNoAuth` | `status` 优先生效，只返回 `frozen` | 无 | 响应状态全为 `frozen` |
| S4 | 请求体 `includeStatuses=["enabled","frozen","disabled"]`，`status=""` | 查询 `pageUserNoAuth` | 只返回 `enabled` 和 `frozen`，不返回 `disabled` | 非法状态被忽略或拒绝，具体实现采用忽略非白名单值 | 响应状态无 `disabled` |
```

来源：

- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/真实目录单个添加药品查询逻辑调整/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/线上线下问诊引用历史处方增加频次带入逻辑/design-codex.md`
- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/报销数据导入增加弃用用户校验/design-codex.md`

## 13. 配套文件引用

模板：

```md
## 配套文件

- 核心认知：`core-cognition.md`
- Foundation Gate：`artifacts/foundation-gate.md`
- 主时序图：`artifacts/<main-sequence>.puml`
- 细分时序图：`artifacts/<side-sequence>.puml`
- API 定义：`artifacts/<module>-openapi.yaml`
- DDL：`artifacts/<module>-ddl.sql`
- 联调前置条件：`artifacts/<topic>-integration-prerequisites.md`

本设计消费模块：

1. <模块 A>
2. <模块 B>
```

时序图约束示例：

```md
时序图约束：

- 查询链路按药房端后台直连 `yt-pha-opm-server` 的 `POST /r/310000/80200/dp/525` 绘制，不包含 `yt-plat` 代理层。
- 单个新增与简化批量导入的医保码补齐口径保持一致，命中标品后直接采用标品第一条的医保码和标准字段。
- `copyDrugCatalogue` 和 `initDpDrugCommons` 为保存成功后的既有后置动作，时序图保留，不展开内部实现。
```

来源：`harness-hh/spec/v1-39-0-version-iteration/specs/v2/真实目录单个添加药品查询逻辑调整/design-codex.md`

## 14. 接口与验收

模板：

```md
## 接口与验收

接口增量：

| 字段 | 位置 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| `<field>` | `<Request/Response/Path/Header>` | `<type>` | <是/否> | <说明> |

验收用例：

1. <场景编号>：<输入> → <输出>。
2. <场景编号>：<输入> → <输出>。

回归验收：

1. <历史接口或历史行为不变>。
2. <旧链路不受新增字段影响>。
```

接口增量示例：

```md
| 字段 | 位置 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| `includeStatuses` | `OauthSysUserDto` 请求体 | `List<String>` | 否 | 状态集合查询字段；支持 `enabled`、`frozen` |
```

回归验收示例：

```md
回归验收：

- 报销导入中，冻结用户仍可通过 `ProjectFeeDao.selectFeeDeptUserDtoListV2` 的公司和报销人名称匹配。
- 组织结构管理保存和编辑冻结用户机构关系不被后端拒绝。
```

来源：`harness-hh/spec/v1-39-0-version-iteration/specs/v2/报销数据导入增加弃用用户校验/design-codex.md`

## 15. 风险、待确认和受影响资产

模板：

```md
## 风险、待确认和受影响资产

风险：

1. <风险和影响>。
2. <风险和影响>。

待确认：

1. <待确认项；若无则写“无阻塞待确认”>。

受影响资产：

- 后端：<仓库 / Controller / Service / DAO / DTO>
- 前端：<页面 / 调用点>
- 数据库：<表 / DDL / 配置项>
- 外部依赖：<MQ / 微信 / HIS / 网关 / Nacos / Redis>

计划同步：

<PLAN_PATH 不存在时写“不涉及既有计划同步”；存在时写是否影响任务、文件范围、测试或顺序。>
```

风险示例：

```md
风险：

- 若调用方误传 `includeStatuses` 包含 `disabled`，必须确保不会扩大到删除用户。
- 若前端将新增字段用于所有用户下拉，会导致默认下拉出现冻结用户；前端应只在需要一次查询正常和冻结用户的场景传该字段。

待确认：

- 无阻塞待确认。

受影响资产：

- 后端：`yt-plat-oauth` 用户查询 DTO 和 Mapper。
- 前端：合同管理中需要一次查询正常和冻结用户的调用点。
- 数据库：无结构变更。
```

最终一致性风险示例：

```md
1. 医保助手对“处方已使用”的识别需要基于 7104 返回内容稳定匹配，建议使用结构化错误码；若只有文本，先匹配包含“处方已使用”。
2. 已经在途的上传结果可能在 type=1 任务终止后仍回传成功，PLAT 补偿重试必须继续检查医保记录。
3. 顶层补偿最多 3 次，每次 30 秒；超过后仍未形成医保成功记录，认为没有需要 7104 作废的医保处方，后续由人工排查日志。
```

来源：

- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/报销数据导入增加弃用用户校验/design-codex.md`
- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`

## 16. 复杂度选择规则

按需求复杂度选择章节，不凑模板：

| 场景 | 必写章节 | 配套文件 |
| --- | --- | --- |
| 单接口工具能力 | 背景、设计决策、接口设计、实现落点、调用链路、验证 | 可只写 `core-cognition.md`，必要时补 OpenAPI |
| 兼容字段扩展 | 背景、范围、兼容约束、方案比选、功能点、接口验收、风险 | `foundation-gate.md`、API 契约 |
| 查询/保存链路改造 | 背景、范围、约束、核心认知、方案比选、总体设计、功能点、受影响资产 | `foundation-gate.md`、时序图、API 契约 |
| 存储或索引变更 | 以上章节 + 存储设计、DDL、上线顺序、回滚和 EXPLAIN 验证 | `ddl.sql`、存储说明 |
| 跨系统流程 | 配套文件、核心认知引用、系统职责、时序图、状态、联调前置条件、风险 | 时序图、OpenAPI、DDL、联调前置条件 |
| 最终一致性 / MQ | 系统职责、事件契约、幂等、重试、补偿、失败终态、对账 | 事件契约、时序图、一致性说明 |

单接口工具能力示例结构：

```md
## 0. 文档目的
## 1. 设计决策
## 2. 接口设计
## 3. 实现落点
## 4. 调用链路
## 5. 数据库配置
## 6. 验证
```

来源：`harness-hh/spec/wechat-menu-delete/specs/design-codex.md`

## 17. 完成前自检

```md
## 自检

1. 主文档是否引用 `core-cognition.md`，且没有复制核心认知全文。
2. `foundation-gate.md` 是否记录代码库、数据库、依赖环境证据。
3. API、时序图、DDL/存储和一致性方案是否在 Gate 和主文档中对齐。
4. 功能点是否都有 `given / when / then / exception / verify`，且 `then` 和 `verify` 可一一对应。
5. 无新增 DDL 时是否明确写“无 DDL 变更”，而不是留空。
6. 真实环境证据不足时，状态是否使用 `pending-human-confirmation` 或明确保留风险，未硬写 `confirmed`。
7. 方案比选是否写明推荐方案和不采用原因。
8. 风险、待确认、受影响资产和计划同步是否有落点。
9. OpenAPI 是否只描述 Controller 契约，未混入 service、DAO、SQL 和消息链路。
10. SQL 示例是否包含库名；DDL 是否写上线顺序、回滚策略和验证方法。
```
