# Foundation Gate 模板

## 1. 用途

本模板用于编写 `artifacts/foundation-gate.md`。

适合写入：

- 总门禁状态
- 模块总览
- 真实代码库证据
- 真实数据库证据
- 真实依赖环境证据
- API、时序图、DDL、数据一致性对齐情况
- 阻塞项与人工确认结论

不适合写入：

- 方案比选细节
- 大段业务推导
- 完整 OpenAPI 字段
- 完整 DDL 内容
- 已在主文档展开的验收叙事

## 2. 文档模板

以下模板建议直接复制为 `artifacts/foundation-gate.md` 后填写。

```md
# Foundation Gate

## 1. 总门禁状态

- 任务：`<task-name>`
- 核心认知：[`../core-cognition.md`](../core-cognition.md)，版本 `<Vfinal>`
- 状态：`draft / pending-human-confirmation / confirmed / confirmed-with-risk / blocked`
- 保留风险模块：<模块列表>
- 人工确认人：<name>
- 确认时间：<YYYY-MM-DD>

## 2. 功能模块总览

| 模块 | 功能点 | API | 时序图 | DDL/存储 | 一致性方案 | 状态 | 阻塞 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| <模块> | <功能点> | `<METHOD /path>` | [`<file>.puml`](./<file>.puml) | <DDL 或“无新增 DDL”> | <一致性摘要> | `<status>` | <阻塞项> |

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

- 结论：`confirmed / confirmed-with-risk / blocked`
- 意见：
```

## 3. 准确示例

以下示例基于历史 spec 抽象，只压缩写法，不改业务事实。

### 3.1 总门禁状态示例

```md
## 1. 总门禁状态

- 任务：`v1-5-0-seven-hospitals`
- 核心认知：[`../core-cognition.md`](../core-cognition.md)，版本 `Vfinal-confirmed-with-risk`
- 状态：`confirmed-with-risk`
- 保留风险模块：整合建档、挂号截止查询、配置中心参数读取
- 人工确认人：胡先生
- 确认时间：2026-05-08
```

适用说明：

- `confirmed-with-risk` 适合“代码、表结构、主链路已核，但真实参数、网关、Redis、Nacos、联调数据仍待补证据”的情况。

来源实践：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/artifacts/foundation-gate.md`

### 3.2 模块总览示例

```md
| 模块 | 功能点 | API | 时序图 | DDL/存储 | 一致性方案 | 状态 | 阻塞 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 挂号截止查询 | 查询当前是否已超过挂号截止时间 | `POST /r/{hosId}/60070/151` | `reg-deadline-check-sequence.puml`、`reg-deadline-window-decision-sequence.puml` | 不新增 DDL；复用机构参数持久化模型和参数缓存 | 只读机构参数；非法配置失败；无持久化变化 | `confirmed-with-risk` | 真实 Redis/Nacos/网关/BFF 和机构参数仍需集成验收 |
```

来源实践：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/artifacts/foundation-gate.md`

### 3.3 模块边界示例

```md
### 3.1 模块边界

- 代码库证据：
  - `RegController`：未发现新增 `60070/151` 入口。
  - `OldRegController#judgeRegs`：历史挂号截止拦截语义参考。
- 数据库证据：
  - 已核对 `config_center_param`、`config_center_config` 字段。
  - `regDeadlineTimeLimit` 当前未查到参数定义或机构值。
- 依赖环境证据：
  - `RegCommonUtil#getRestCommonParam` 先读 Redis 参数缓存，miss 后回源数据库。
- 模块职责：只读机构参数和服务端当前时间，返回 `overDeadline / inLimitWindow`。
- 不负责：不替代挂号提交校验，不创建挂号单。
```

来源实践：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/artifacts/foundation-gate.md`

### 3.4 API 清单示例

```md
| 调用端/tag | 功能点 | Method | Path | 请求 VO | 响应 VO | 副作用 | 幂等 | 状态 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| H5端 / `seven-hospitals-reg` | REG 查询当前是否已超过挂号截止时间 | POST | `/r/{hosId}/60070/151` | 无 requestBody；`hosId` 为历史路由维度参数 | `ResultVo<RegDeadlineCheckResult>` | 查询本身不写业务库；缓存 miss 时可能写入参数缓存 | 查询幂等；同一参数和同一时间下结果确定 | `confirmed-with-risk` |
```

来源实践：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/artifacts/foundation-gate.md`

### 3.5 数据一致性方案示例

```md
| 场景 | 范围 | 强一致/最终一致 | 事务边界 | 幂等/补偿 | 对账/重试 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |
| 新增就诊人成功，建档失败 | 就诊人已生成，院内档案未生成有效卡 | 允许部分成功，原合同用 `PAT_SUCCESS_RECORD_FAIL` 暴露 | 新增就诊人不可因建档失败自动回滚 | 返回 `patId` 供前端识别；后续是否补建档待业务确认 | 无自动重试；人工或前端重提策略待确认 | `confirmed-with-risk` |
```

来源实践：

- `harness-hh/spec/v1-5-0-seven-hospitals/specs/artifacts/foundation-gate.md`

## 4. 自检清单

1. 总门禁状态与模块状态一致，没有一个写 `blocked`、总状态却写 `confirmed`。
2. 每个模块都写了代码库证据、数据库证据、依赖环境证据。
3. 无新增 DDL 的模块明确写“无新增 DDL”，不留空。
4. API、时序图、DDL、数据一致性四类信息至少各有一个落点。
5. 真实环境证据不足时，状态使用 `pending-human-confirmation`、`confirmed-with-risk` 或 `blocked`，不硬写 `confirmed`。
