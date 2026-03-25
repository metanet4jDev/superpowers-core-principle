---
name: gating-existing-changes
description: 用于设计或规划存量系统改动前，先卡住关键前提，避免过早收敛。
---

# 存量变更闸门

## 何时使用
- 可能改行为、接口、状态、存储或协同方
- 可能跨仓库或跨模块
- 成功标准不能只靠代码直接判断

不用于：只读分析，或纯机械重构。

## 使用方法
在写设计文档、方案、计划、代码前，先输出：

```text
Change Gate
- target: confirmed | unconfirmed | n/a
- scope: confirmed | unconfirmed | n/a
- compatibility: confirmed | unconfirmed | n/a
- persistence: confirmed | unconfirmed | n/a
- coordination: confirmed | unconfirmed | n/a
- success_criterion: confirmed | unconfirmed | n/a
```

含义：
- `target`：止血还是最终方案
- `scope`：单仓库还是跨仓库
- `compatibility`：能不能改旧接口、旧返回、旧行为
- `persistence`：能不能改表、字段、存储
- `coordination`：前端或下游要不要一起改
- `success_criterion`：什么才算成功

## 规则
- 只要有 `unconfirmed`，按头脑风暴方式一次只问一个关键问题，然后停止。
- 未全部确认前，不写设计文档、计划、代码。
- 真不适用就写 `n/a`。
- 拿不准就写 `unconfirmed`，不要猜。
