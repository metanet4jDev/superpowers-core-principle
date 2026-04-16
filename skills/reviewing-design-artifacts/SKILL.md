---
name: reviewing-design-artifacts
description: Use only when the user explicitly asks to check, validate, or review core-cognition and detailed-design artifacts together.
---

# 检查设计产物

## Purpose

在用户明确要求后，整体检查独立核心认知文档和详细设计产物是否满足要求。只报告问题和证据；不要自动修改，除非用户明确要求修正。

硬规则：

- 不自动触发。写完设计、计划或代码后都不能顺手执行本技能。
- 核心认知和详细设计固定是两个文档：`CORE_COGNITION_PATH` 与 `DESIGN_DOC_PATH`。
- 核心认知检查只看核心认知完整性；详细设计检查只看详细设计和配套文件。
- 跨文档检查只判断引用、去重和一致性，不把一个文档的职责转嫁给另一个文档。
- findings first；每条问题必须有位置、证据、违反规则、修正方向、验证方式。

## When to Use

Use only when the user explicitly says one of:

- 检查核心认知和详细设计。
- 审查/验证设计产物。
- 对核心认知、详细设计和配套文件做整体 review。
- 判断设计是否可以进入计划或实现。

Do not use when:

- 用户只是要求写核心认知或详细设计。
- 用户只要求验证核心认知；用 `core-principle/validation-checklist.md`。
- 用户只要求验证详细设计；用 `writing-detailed-design/validation-checklist.md`。
- 用户要求审查计划或代码；使用对应 review 技能。

## Required Inputs and Paths

先用 `superpowers:document-workspace-layout` 解析：

- `CORE_COGNITION_PATH`
- `DESIGN_DOC_PATH`
- `DESIGN_ARTIFACTS_DIR`
- `PLAN_PATH`
- `PLAN_CHANGELOG`

如果缺少 `CORE_COGNITION_PATH` 或 `DESIGN_DOC_PATH`，直接报告阻断问题，不猜测替代文件。

## Review Order

1. **核心认知独立检查**
   使用 `core-principle` 的 `validation-checklist.md`。只检查核心认知的边界、7 维、6 标题、持久化事实和单一事实源。

2. **详细设计独立检查**
   使用 `writing-detailed-design` 的 `validation-checklist.md`。只检查主文档、配套文件、API/事件契约、功能点、验收和 plan drift。

3. **跨文档一致性**
   检查：

   - 详细设计是否引用 `CORE_COGNITION_PATH`。
   - 详细设计是否复制或重定义核心认知事实。
   - 功能点引用的实体、关系、属性、状态是否能回到核心认知定位。
   - 核心认知的功能总览是否能覆盖详细设计的功能点范围。
   - 全局约束是否在详细设计中被消费，而不是被局部规则改写。
   - 配套文件是否被主文档引用，且不与主文档重复定义。
   - 若 `PLAN_PATH` 已存在，详细设计变更是否同步到计划或记录为不影响。

## Severity

- `阻断`：会导致计划、实现、联调、测试或上线方向错误。
- `主要`：缺少关键设计信息，容易造成实现歧义或重复事实源。
- `次要`：组织、引用、命名或局部可读性问题，不改变行为。
- `观察`：非阻断建议或残余风险。

## Output Format

```markdown
## Design Artifact Review

### Findings

1. [阻断|主要|次要|观察] <问题标题>
   - 位置：
   - 证据：
   - 违反规则：
   - 修正方向：
   - 验证方式：

### Coverage

- 核心认知独立检查：
- 详细设计独立检查：
- 跨文档引用：
- 去重与单一事实源：
- 配套文件：
- Plan drift：

### Open Questions

- <需要用户确认的问题；没有则写“无”>

### Summary

- <是否可进入计划 / 是否需修改>
- <剩余风险或未核验项>
```

未发现问题时，明确写“未发现问题”，并说明未核验项。
