# Interface First Review Checklist

仅在本 skill 的 review gate 阶段使用。不要自动扩展到代码实现评审。

本清单只检查：
- 设计文档到 API 契约的承接
- API 契约到 interface 契约的承接
- 是否已经满足“可以开始单接口 TDD”的前置条件

本清单不检查：
- 业务实现质量
- SQL/DAO 细节
- 测试覆盖率
- 代码风格

## 验证步骤

### 1. 事实源是否单一

- 是否已经读取并引用设计文档、核心认知、API 设计规范。
- 每个接口的 path、method、请求、响应、异常语义是否都能回到设计文档定位。
- 是否存在“代码里先拍脑袋定接口，文档再回填”的情况。

### 2. API 契约是否先于实现

- OpenAPI / controller API 定义是否已经落地。
- 是否仍停在契约阶段，没有偷跑业务实现。
- 已上线历史接口若保持原逻辑不变，是否显式写明“不改变历史逻辑”。
- 新增边界是否已写进契约：
  - 字段类型
  - nullable
  - 枚举
  - 空值语义
  - 幂等/副作用
  - 错误态

### 3. API 契约是否可实现

- path、method、请求 VO、响应 VO 是否命名稳定。
- 同一业务对象是否复用同一 schema。
- 金额、状态、时间字段的类型和语义是否一致。
- nullable 字段是否写清何时为 `null`，`null` 表示什么。
- 查询接口是否无副作用；有副作用则是否显式说明。
- 本地契约文件是否能解析。

### 4. interface 契约是否承接 API 语义

- service interface 是否已经定义。
- 必要的 domain service / facade / gateway interface 是否已经定义。
- interface 参数、返回值命名是否承接 API/设计文档语义。
- interface 是否暴露了实现细节、DAO 细节、SQL 细节。
- 是否存在一个 API 对多个 interface 责任不清的情况。

### 5. review gate 是否真的成立

- 是否已明确列出待 review 文件。
- 是否已明确告诉协作者：当前处于 review gate，尚未进入实现。
- 是否已经约束“review 未通过前，不写业务实现”。
- 是否已经约束“review 通过后，一次只推进一个接口”。

### 6. 是否满足进入单接口 TDD 的前提

对每个待实现接口检查：

- 输入边界已确定
- 输出边界已确定
- 错误语义已确定
- 幂等/副作用已确定
- service interface 已确定
- 不需要再靠实现反推契约

只要有一项不成立，就不应进入实现。

## 输出格式

```markdown
## Interface First Review

### Findings

1. [阻断|主要|次要|观察] <问题标题>
   - 位置：
   - 证据：
   - 违反规则：
   - 修正方向：
   - 验证方式：

### Gate Decision

- API 契约：
- interface 契约：
- review gate：
- 是否允许进入单接口 TDD：

### Summary

- <通过 review gate / 不通过 review gate>
```
