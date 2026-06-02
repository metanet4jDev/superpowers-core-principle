# 详细设计模板索引

本目录将详细设计拆成四类模板：

1. 完整设计文档样例：[design-codex-full-example.md](./design-codex-full-example.md)
2. 主设计文档骨架：[design-codex-template.md](./design-codex-template.md)
3. 门禁文档模板：[foundation-gate-template.md](./foundation-gate-template.md)
4. 历史实践示例库：[practice-example-template.md](./practice-example-template.md)

使用顺序：

1. 先看 [design-codex-full-example.md](./design-codex-full-example.md)，按完整设计文档形态建立直觉。
2. 再补齐 `core-cognition.md`。
3. 再写 `artifacts/foundation-gate.md`，确认模块证据、阻塞项、门禁状态。
4. 门禁达到 `confirmed`、`confirmed-with-risk` 或用户明确接受风险后，再写 `design-codex.md`。
5. 时序图、OpenAPI、DDL、契约、联调前置条件拆到配套文件，不在主文档重复展开。
6. 需要更多局部写法时，再查 [practice-example-template.md](./practice-example-template.md)。

最小输出物：

1. `specs/core-cognition.md`
2. `specs/artifacts/foundation-gate.md`
3. `specs/design-codex.md`

编写原则：

1. 核心认知只引用，不复制。
2. 同一事实只定义一次，其他位置只引用。
3. 主文档讲决策、边界、方案、验收和风险。
4. 门禁文档讲证据、可走通性、阻塞项和确认状态。
5. 示例必须基于真实历史 spec，不凭空造接口、表名、状态名。
6. 需要整体结构时，优先参考完整设计文档样例；需要局部段落写法时，参考历史实践示例库。
