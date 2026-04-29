---
name: requesting-code-review
description: 仅在用户明确要求对已完成工作、重大功能或合并前状态执行代码评审时使用。
---

# 发起代码评审

代码评审必须由用户触发。不要在任务、批次、功能或一次实现工作结束后，自动派发 `code-reviewer` 子代理。

允许的行为：

- 告诉用户代码评审是可用的。
- 等待用户明确下达指令，例如 “review this”、“run code review” 或 “检查代码”。
- 在评审反馈返回后，只有当用户要求你处理这些问题时才去修复，除非用户在提出评审请求时已经一并授权你处理反馈。

## 何时使用

仅在用户明确要求代码评审时使用：

- 完成某项任务或某一批工作之后
- 完成某个重大功能之后
- merge 或 PR 之前
- 当前卡住，且用户希望有人从新视角再审一遍

即使符合上述场景，如果用户没有明确要求，也不能使用本技能。

以下情况不要使用：

- 计划里只是写了一个“评审检查点”
- 某个任务刚做完，但用户还没有要求评审
- 你正准备进入下一项任务，只是想顺手自动检查上一项

## 如何发起

1. 获取 git SHAs：

```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

2. 只有在用户明确下达指令后，才派发 `code-reviewer` 子代理。

使用模板：`requesting-code-review/code-reviewer.md`

占位符含义：

- `{WHAT_WAS_IMPLEMENTED}`：本次实现了什么
- `{PLAN_OR_REQUIREMENTS}`：它本来应该做什么
- `{BASE_SHA}`：起始提交
- `{HEAD_SHA}`：结束提交
- `{DESCRIPTION}`：简要摘要

3. 按照用户指示处理评审反馈：

- 严重问题：清楚报告；如果用户要求处理评审结果，则修复
- 重要问题：继续之前先说明影响
- 次要问题：除非用户要求修复，否则记为后续跟进项
- 如果 reviewer 判断有误，要拿证据反驳

## 与工作流的集成

- `subagent-driven-development` 可以停下来提示“现在可以做代码评审”，但未经用户指示不得自动派发评审
- `executing-plans` 可以报告“某个批次已可评审”，但后续动作必须遵从用户指令
- `finishing-a-development-branch` 可以建议在合并前做代码评审，但必须等待用户批准

## 风险信号

绝不要：

- 因为某个任务完成了，就自动派发代码评审
- 把“评审检查点”当作默认授权
- 当用户只要求出评审报告时，自动去修复评审反馈
- 已知存在严重问题，却不告知用户就继续推进

模板见：`requesting-code-review/code-reviewer.md`
