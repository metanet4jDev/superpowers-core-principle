---
name: interface-first-development
description: "Use when implementing a feature from a design document and the work should follow interface-first development: define HTTP API contracts and service interfaces before implementation, stop for review approval, then continue with single-interface TDD delivery."
---

# 面向接口优先开发

## 概览

先定契约，后写实现。

核心规则：
1. 设计文档是事实源
2. 先落 API 契约，再落内部 interface 契约
3. `review gate` 前不写业务实现
4. `review gate` 通过后，按单接口进入 TDD

接口边界没写清，后续实现只会把不确定性埋进代码。

## 何时使用

适用场景：
- 用户明确要求按设计文档推进开发
- 用户希望先定义 API 接口、内部 interface，再做实现
- 用户希望把 `review` 作为实现前门禁
- 任务适合按“一个接口一个接口”推进

不适用场景：
- 纯文档任务，没有代码落点
- 纯重构任务，没有新增或调整接口
- 用户明确要求直接实现，不设 `review gate`

## 开场声明

开始时声明：
`I'm using the interface-first-development skill to define API and interface contracts before implementation.`

如果同时写文档或契约说明，再声明：
`I'm using the caveman skill and the writing-clearly-and-concisely skill for concise, readable documentation.`

如果进入实现阶段，再声明：
`I'm using the test-driven-development skill for single-interface implementation after review approval.`

## 流程

### 第 0 步：锁定事实源

1. 读取设计文档、核心认知、相关 API 设计规范
2. 抽出本次涉及的接口清单
3. 对每个接口记录：
   - path / method
   - 请求 VO
   - 响应 VO
   - service interface
   - 幂等、副作用、异常语义
4. 如果设计文档对关键行为未写清，先追问或补齐，不要跳到实现

### 第 1 步：先写 API 契约

先改或新增 OpenAPI、controller API 定义。

要求：
- 契约命名对齐真实代码
- 字段类型、`nullable`、枚举、空值语义写清
- 金额、时间、状态字段口径写清
- 历史接口若保持原逻辑不变，必须显式写明
- 不要一边补契约一边写业务实现

输出物示例：
- `api-contract.openapi.yaml`
- controller 接口签名
- 请求/响应 DTO 草案

### 第 2 步：再写 interface 契约

API 契约稳定后，再定义内部 interface。

最少包括：
- service interface
- 关键 facade / domain service interface
- 必要的 repository / gateway interface 边界

要求：
- interface 先表达能力，再讨论实现
- 参数和返回值命名必须承接 API/设计文档语义
- 不要让 controller 直接把未定型的实现细节泄漏到 interface

### 第 3 步：停在 review gate

API 契约和 interface 契约写完后，停止。

此时必须做：
1. 汇总已定义接口
2. 标出待 review 文件
3. 明确告诉用户：当前处于 `review gate`，尚未进入实现
4. 使用 `interface-review-checklist.md` 检查是否真的满足进入实现前门禁

未获 `review` 通过，且用户未明确豁免前：
- 不写业务实现
- 不补“顺手能写”的 service 逻辑
- 不写“先占位”的 mapper/SQL

`review gate` 输出至少包含：
- 待 review 文件列表
- 每个接口的 API 契约状态
- 每个接口的 interface 契约状态
- 是否允许进入单接口 TDD 的结论

推荐输出模板：
```md
当前处于 review gate，尚未进入实现。

待 review 文件：
- ...

接口状态：
- `POST /...`：API 契约已落地，service interface 已落地
- `GET /...`：API 契约已落地，repository interface 待补充

结论：
- 暂不允许进入单接口 TDD
```

## Review Gate 通过后的实现方式

`review gate` 通过后，按单接口串行推进。

一次只推进一个接口：
1. 选一个接口
2. 先写该接口失败测试
3. 只写让该接口通过的最小实现
4. 跑验证
5. 再进下一个接口

不要这样做：
- 多个接口一起并行实现
- 所有 DTO 先堆完，再统一补测试
- controller、service、mapper、SQL 全部铺开后再回头补契约

## 单接口 TDD 模板

对每个接口重复：

1. RED
   - 写该接口的失败测试
   - 失败原因必须是“功能未实现”或“契约未满足”
2. GREEN
   - 写最小实现让当前接口通过
   - 只改当前接口真正需要的代码
3. REFACTOR
   - 清理重复
   - 保持契约不漂
4. VERIFY
   - 重新跑该接口测试
   - 跑相关回归测试

## 决策顺序

固定顺序：

`设计文档 -> API 契约 -> interface 契约 -> review gate -> 单接口 TDD -> 下一个接口`

不要倒序。想“先写一点实现再补契约”，说明边界还没想清。

## 常见错误

### 1. 用实现倒逼契约

错法：
- 先写 service / SQL
- 再把现有实现包装成 API

正法：
- 先定输入、输出、异常、状态语义
- 再实现

### 2. review gate 形同虚设

错法：
- 契约写完后顺手把核心逻辑也写了

正法：
- 契约阶段结束就停
- 等 `review` 结果或用户明确豁免

### 3. 一次推进多个接口

错法：
- 想“反正都在同一模块，一起写更快”

正法：
- 一次一个接口
- 每个接口独立 RED-GREEN-REFACTOR

### 4. API 和 interface 命名漂移

错法：
- OpenAPI 叫一个名字
- Java interface 叫另一个语义
- DTO 字段再来第三套

正法：
- 设计文档语义一套到底

### 5. 契约阶段偷带实现细节

错法：
- 在 interface 里提前暴露 mapper、表结构、第三方返回体细节

正法：
- interface 只表达业务能力和边界
- 实现细节留到实现阶段决定

## 完成条件

一次完整推进，分两段完成。

### 第一段：契约阶段完成

满足全部条件才算完成：
- API 契约已落地
- interface 契约已落地
- 变更点已汇总
- 已明确进入 review gate

### 第二段：实现阶段完成

满足全部条件才算完成：
- review 已通过或用户明确豁免
- 每个接口按单接口 TDD 完成
- 测试通过
- 契约与实现一致

## 记住

- 先定义接口，不先写实现
- 先过 `review gate`，不偷跑
- 一次一个接口，不批量铺开
- TDD 只在 `review` 通过后进入
- 契约是边界，不是事后记录
