# Design Codex 模板

## 1. 用途

本模板用于编写 `design-codex.md`。

适合写入：

- 背景和目标
- 范围和非目标
- 兼容与上线约束
- 核心认知引用
- 方案比选
- 总体设计
- 功能点详细设计
- 验收、风险、待确认项、受影响资产

不适合写入：

- 全量表结构
- OpenAPI 全量字段
- 大段 SQL
- 完整时序图源码
- 已在 `core-cognition.md` 中稳定存在的大段实体定义

## 2. 文档模板

以下模板建议直接复制为 `design-codex.md` 后填写。

```md
# <业务主题>详细设计

- 日期：<YYYY-MM-DD>
- 任务：`<task-name>`
- 适用范围：`<repo-a>`、`<repo-b>`
- 文档口径：本文保留设计决策、边界、方案、验收和风险；稳定事实以 [`./core-cognition.md`](./core-cognition.md) 为唯一主落点。

## 配套文件

1. 核心认知：[`core-cognition.md`](./core-cognition.md)
2. 地基门禁：[`artifacts/foundation-gate.md`](./artifacts/foundation-gate.md)
3. 时序图：[`artifacts/<main-sequence>.puml`](./artifacts/<main-sequence>.puml)
4. API 契约：[`artifacts/<module>-openapi.yaml`](./artifacts/<module>-openapi.yaml)
5. 存储设计：[`artifacts/<module>-ddl.sql`](./artifacts/<module>-ddl.sql)
6. 联调前置条件：[`artifacts/<topic>-integration-prerequisites.md`](./artifacts/<topic>-integration-prerequisites.md)

## 1. 背景和目标

<说明当前问题、历史歧义、这次设计要解决什么，以及成功标准。>

## 2. 范围和非目标

### 2.1 范围

1. <范围 1>
2. <范围 2>

### 2.2 非目标

1. <非目标 1>
2. <非目标 2>

## 3. 执行、上线和兼容约束

1. <约束 1>
2. <约束 2>

## 4. 核心认知引用

- 路径：[`./core-cognition.md`](./core-cognition.md)
- 版本：`<Vfinal or Vfinal-confirmed-with-risk>`

本设计依赖的核心结论：

1. <结构或系统边界>
2. <持久化模型或状态>
3. <全局约束或兼容约束>

## 5. 方案比选

| 方案 | 描述 | 优点 | 缺点 | 结论 |
| --- | --- | --- | --- | --- |
| A | <方案 A> | <优点> | <缺点> | 不采用 |
| B | <方案 B> | <优点> | <缺点> | 采用 |

推荐方案：`<方案名>`。

## 6. 总体设计

### 6.1 系统职责 / API 全景 / 模块分工

| 系统/模块 | 职责 |
| --- | --- |
| <系统 A> | <职责> |
| <系统 B> | <职责> |

### 6.2 关键实现落点

| 能力 | 目标模块 | 目标入口 | 复用锚点 |
| --- | --- | --- | --- |
| <能力> | `<module>` | `<controller/service>` | `<历史锚点>` |

### 6.3 实现约束

1. <约束 1>
2. <约束 2>

## 7. 功能点详细设计

### 7.1 功能点：<功能点名称>

功能：

- 动作：<动作>
- 目的：<目的>
- 作用对象：<实体/接口/表/消息>
- 执行路径：<主路径>
- 失败路径：<失败路径>
- 终止条件：<终止条件>
- 序列图：[`artifacts/<xxx>.puml`](./artifacts/<xxx>.puml)

参与实体：

- `<实体 1>`
- `<实体 2>`

界限：

| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| F1 | <前置条件> | <触发动作> | <结果> | <异常口径> | <验证方式> |
| F2 | <前置条件> | <触发动作> | <结果> | <异常口径> | <验证方式> |

## 8. 配套文件

- 核心认知：`core-cognition.md`
- 地基门禁：`artifacts/foundation-gate.md`
- 时序图：`artifacts/<xxx>.puml`
- OpenAPI：`artifacts/<xxx>-openapi.yaml`
- DDL：`artifacts/<xxx>-ddl.sql`

## 9. 接口与验收

1. <关键验收场景 1>
2. <关键验收场景 2>

## 10. 风险

1. <风险 1>
2. <风险 2>

## 11. 待确认项

1. <待确认 1>
2. <待确认 2>

## 12. 受影响资产

### 12.1 现有方法

- `<Class#method>`

### 12.2 建议新增或扩展

- `<Class/DTO/Service>`

### 12.3 不应新增

- `<不应新增的接口/表/旁路入口>`
```

## 3. 准确示例

以下示例基于历史 spec 抽象，只压缩写法，不改业务事实。

### 3.1 背景和目标示例

```md
## 1. 背景和目标

医生在 `/d/90120/930` 作废处方后，平台处方已经 `ABOLITION=1`，但医保上传链路仍可能继续执行，后续 `olt_prescription_med_insure.rx_stas_codg='1'`，医保侧处方生效并发生结算。

本次目标：

1. 医生作废前先由医保助手尝试医保 7104 作废。
2. 平台处方作废后通过 RocketMQ 延时消息做医保作废补偿。
3. 作废后终止处方上传任务，避免上传与作废并发窗口继续放大。
```

来源实践：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`

### 3.2 范围和非目标示例

```md
### 2.1 范围

1. 在 `yt-med-pat-server` 新增 `POST /r/{srcId}/50004/141` 整合建档接口。
2. 在 `yt-med-reg-server` 新增 `POST /r/{hosId}/60070/151` 挂号截止查询接口。
3. 在 `yt-med-cfg-server` 新增 `POST /r/{srcId}/50001/312` 配置中心参数读取接口。

### 2.2 非目标

1. 不修改历史接口 `50004/140`、`50004/300`、`60070/150`。
2. 不新增数据库表、DDL、索引或迁移脚本。
3. 不把 service、DAO、SQL、消息链路写进 OpenAPI 契约。
```

来源实践：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/design-codex.md`

### 3.3 方案比选示例

```md
| 方案 | 描述 | 优点 | 缺点 | 结论 |
| --- | --- | --- | --- | --- |
| A | 平台先作废，再完全依赖 MQ 补偿医保作废 | 对 RP 改动少 | 医保已使用时仍会本地作废，用户无法被即时拦截 | 不采用 |
| B | 医保助手先尝试医保作废；成功才调用 RP 作废；“处方已使用”拦截；其他失败走 RP 作废和 MQ 补偿 | 能拦截医保已使用，兼容上传并发窗口，符合最终一致性目标 | 需要 RP、PLAT、医保助手协同 | 采用 |
| C | 医保作废必须成功才允许平台作废，其他失败全部阻断 | 一致性最强 | 上传中场景会误阻断平台作废，无法解决并发窗口 | 不采用 |
```

来源实践：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`

### 3.4 功能点边界表示例

```md
| 编号 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| N3 | 报价请求带 `historyPscriptId`、`historyPMedDetailId`，且历史处方 `HOS_ID != 当前 hosId` | 调用 `/getDrugQuote` | 频次和用法都置空；不再继续做历史映射和目录默认值回退 | 无 | 核对返回字段为空，且不会命中目录默认值 |
| N4 | 历史处方属于当前机构，历史频次/用法都能映射当前机构字典 | 调用 `/getDrugQuote` | 优先返回历史映射值；`frequencyQty`、`presSustainedDays` 继续沿历史值透传 | 无 | 核对返回编码和名称来自当前机构字典，而不是原始历史字符串 |
| N5 | 历史处方属于当前机构，但历史频次或历史用法无法映射当前字典 | 调用 `/getDrugQuote` | 对应字段回退到当前命中目录默认值；仍按字段独立判断 | 目录无默认值或默认值不可用时，对应字段置空 | 分别构造“频次失败、用法成功”和“用法失败、频次成功”两套用例 |
```

来源实践：

- `harness-hh/spec/v1-39-0-version-iteration/specs/v2/线上线下问诊引用历史处方增加频次带入逻辑/design-codex.md`

### 3.5 一致性设计示例

```md
### 6.3 PLAT 补偿消息消费

功能：在作废后终止上传任务，并在医保上传成功时创建医保作废任务。

| 场景 | given | when | then | exception | verify |
| --- | --- | --- | --- | --- | --- |
| 1 | 平台处方已作废，医保记录已上传成功 | 消费补偿消息 | type=1 上传任务置作废；幂等创建 type=2 作废任务 | 已存在 type=2 时不重复插入 | `med_prescription_upload_task` 有且仅有一个 type=2 待处理任务 |
| 2 | 平台处方已作废，医保记录尚未上传成功，重试次数未达 3 | 消费补偿消息 | type=1 上传任务置作废；发送下一轮 30 秒延时消息 | MQ 发送失败按现有 Producer 失败处理 | 下一轮消息 `retryTimes+1` |
```

来源实践：

- `harness-hh/spec/prescription-cancel-medins-consistency/specs/design-codex.md`

## 4. 自检清单

1. 已引用 `core-cognition.md` 和 `artifacts/foundation-gate.md`。
2. 没有复制核心认知大段正文。
3. 每个功能点都包含 `given / when / then / exception / verify`。
4. 风险和待确认项已显式写出。
5. 主文档中的 API、时序图、DDL、契约、一致性描述互不冲突。
